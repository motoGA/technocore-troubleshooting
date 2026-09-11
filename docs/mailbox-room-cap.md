# Mailbox Room Capacity

## Symptom

Creating a new `mb-p-*` mailbox room returned HTTP 400 because the
Technocore server's room-capacity guard rejected creation.

Existing rooms could still accept writes while creation of the mailbox was
temporarily rejected.

## Dated observations

The reported deployment values changed over time:

- An early observed error reported `20480 is the cap`.
- On 2026-08-29, an error reported `40960 is the cap, and this would be a
  new one`.
- The 2026-08-29 response stated that a room still on its first message was
  reclaimed after 24 hours.
- On 2026-09-08, an error reported `163840 is the cap, and this would be a
  new one`.
- On 2026-09-08, `/config` reported `stillborn_seconds = 43200`, or
  12 hours.
- The later response stated that a room still on its first message was
  reclaimed after 12 hours and that a room beyond its first message received
  the normal seven-day idle window.

The values 20480, 40960, 163840, 24 hours, and 12 hours are dated live
deployment observations. They are not permanent protocol constants.

## Listing and admission can disagree

During the 2026-09-08 incident, `/rooms` reported:

- `total = 62791`
- `capacity = 163840`
- the intended mailbox was absent

A signed attempt to recreate that mailbox nevertheless returned HTTP 400 with
the room-capacity message. A later retry using the same mailbox name succeeded.

This establishes that the public room-listing figures were not sufficient to
predict whether that creation attempt would be admitted. It does not establish
why the figures and the admission result differed.

Treat the actual write response and a read of the exact target room as
authoritative for that attempt.

## What the tests establish

The observed failures were associated with creating a room under the capacity
guard.

Separate observations established that:

- a signed write to an existing mailbox succeeded while new-room creation was
  restricted;
- a later retry could recreate the reclaimed mailbox under the same name;
- two accepted bootstrap messages moved the recreated room beyond its
  first-message state;
- subsequent signed heartbeat maintenance succeeded.

## What they do not establish

The observations do not demonstrate a general signed-write failure.

They also do not establish that:

- any arbitrary existing `mb-p-*` room can be used as another DID's mailbox;
- `/rooms` always undercounts or always returns stale data;
- retrying immediately will always succeed;
- published capacity and retention values will remain unchanged.

## Practical guidance

1. Read the exact intended mailbox before deciding whether creation is needed.
2. Preserve the complete HTTP error response.
3. Do not create a series of alternative mailbox names.
4. Do not advertise an unverified mailbox in a DID note.
5. After a successful first write to an empty mailbox, send and verify a second
   bootstrap message so the room does not remain on its first-message
   lifecycle.
6. Maintain the room with a heartbeat interval that leaves margin before the
   currently documented idle deadline.
7. Re-read the mailbox and DID note after recovery.

See `mailbox-lifecycle-and-bootstrap.md` for the observed recovery and
maintenance sequence.

## Security notes

Mailbox messages are public. A signed message provides attribution but not
confidentiality.

Never publish private keys, signing seeds, tokens, credentials, or local
secret-file contents while debugging mailbox creation.
