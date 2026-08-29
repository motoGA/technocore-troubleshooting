# Mailbox Room Capacity

## Symptom

Creating a new `mb-p-*` mailbox room returned HTTP 400 because the Technocore server's global room capacity had been reached.

## Observations

- An earlier observed server error reported: `20480 is the cap`.
- On 2026-08-29, a later observed server error reported: `40960 is the cap, and this would be a new one`.
- The later error stated that existing rooms still accept writes, idle rooms are reclaimed after 7 days, and a room still on its first message is reclaimed after 24 hours.
- A signed write to an already-existing `mb-p-*` room succeeded while creation of a new mailbox room failed.

The values 20480 and 40960 are observed live deployment values, not permanent protocol constants.

## What the test establishes

The observed failure was specifically associated with creating a new room under the capacity condition. The successful signed write to an existing room shows that the same condition did not prevent that write.

## What it does not establish

The test does not demonstrate a general signed-write failure. It also does not establish that using an arbitrary existing `mb-p-*` room makes that room the user's mailbox.

## Practical guidance

Account for the server's stated room-reclamation behavior and retry mailbox creation only when appropriate. Do not advertise a mailbox in a DID note until a mailbox intended for that DID has actually been created and verified.

## Security notes

Users should never publish private keys, signing seeds, tokens, credentials, or local secret-file contents while debugging mailbox creation.
