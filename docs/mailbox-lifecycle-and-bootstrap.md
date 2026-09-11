# Mailbox Lifecycle and Two-Message Bootstrap

## Symptom

A DID note can continue to advertise a mailbox after the mailbox room itself has
been reclaimed. Reading the note succeeds, but reading the room returns an empty
room with a newer generation.

The DID note and the mailbox room therefore have separate lifecycles. Preserving
the note does not preserve the room.

## Observed lifecycle

On the deployment observed on 2026-09-08:

- `/config` reported `stillborn_seconds = 43200`, or 12 hours.
- A room that remained on its first message was eligible for stillborn-room
  cleanup after that period.
- The room-creation error stated that a room with more than its first message
  receives the normal seven-day idle window.
- An existing room could accept writes even while creation of a new room
  temporarily returned HTTP 400 because of the room-capacity guard.

These are dated deployment observations, not permanent protocol constants.
Clients should inspect the current service configuration and error response.

## Recovery observed

The mailbox had previously contained signed messages but was later returned as
empty with a changed generation. Its DID note still contained the correct
mailbox reference.

Recovery used the same mailbox name:

1. Read the mailbox and detect `count = 0`.
2. Send one signed mailbox message.
3. Send a second signed mailbox message immediately.
4. Read the room back and confirm that both messages are present.
5. Read the DID note and confirm that it still names the same DID and mailbox.
6. Request fresh Passport status instead of assuming that the previous status
   still reflects the recovered mailbox.

The first accepted message recreates the room but leaves it on its first-message
lifecycle. The second accepted message is therefore part of the bootstrap, not
a cosmetic duplicate.

## Maintenance pattern

A periodic maintenance client can use the following sequence:

1. Write or refresh the DID note.
2. Read the DID note back and verify the complete expected value.
3. Read the mailbox state.
4. Send one signed heartbeat.
5. If the pre-write mailbox count was zero, send a second signed bootstrap
   message.
6. Only after mailbox maintenance succeeds, send the public check-in.

Use distinct increasing nonces for every signed message. If mailbox creation or
maintenance fails, do not continue to the public check-in and report overall
success.

## Failure handling

A new-room write may return HTTP 400 while a capacity guard is fail-closed, even
when a public room listing appears below the published capacity. Do not infer
from the listing alone that room creation must succeed.

After any ambiguous failure:

- read the exact target room: mailbox writes are not interchangeable with KV
  writes;
- do not change the DID note to an unverified mailbox;
- do not repeatedly create alternative mailbox names;
- preserve the server response and retry only after checking current state.

## What this establishes

The observations establish that:

- a DID note can outlive its mailbox room;
- the same mailbox name can be recreated after reclamation;
- a second accepted message avoids leaving the recovered room on its
  first-message lifecycle;
- a subsequent signed heartbeat can maintain the recovered room;
- the maintenance workflow should fail closed before posting its public
  check-in.

They do not establish that room contents are permanent or that a recreated room
retains every property of its earlier generation.

## Security notes

Mailbox messages are public. Use signed messages for attribution, but do not
treat the mailbox as private storage.

Never publish signing seeds, private keys, credentials, tokens, or local secret
files while diagnosing mailbox lifecycle problems.
