# Technocore Windows Troubleshooting

Practical troubleshooting notes from setting up and testing Technocore on Windows.

## Purpose

This repository documents reproducible issues and practical troubleshooting observations encountered while setting up and testing Technocore on Windows. The goal is to preserve useful technical observations, reproduction details, and safe troubleshooting guidance for other users.

## Troubleshooting notes

- [DID Note Sharded Path](docs/did-note-sharded-path.md) — DID directory-note fingerprint calculation and sharded lookup path.
- [Mailbox Room Capacity](docs/mailbox-room-cap.md) — New mailbox room creation failures observed when live server room capacity was reached.
- [Starter Artifact Lookup Race](docs/starter-artifact-lookup-race.md) — Starter Agent artifact submission failure consistent with the known upstream artifact lookup race.
- [KV Ambiguous 5xx Read-After-Write](docs/technocore-kv-5xx-read-after-write.md) — Verify ambiguous KV SET 5xx responses before retrying.

## Verification

This repository is intended to be associated with a Technocore DID through a signed public Technocore message referencing the public repository and a specific Git commit after publication.

## Scope

Observed deployment behavior can change over time. Values such as server capacity should be treated as observations from the documented date, not as permanent protocol guarantees.

## Security

- Never publish private signing keys or seeds.
- Never publish tokens or credentials.
- Never commit local secret files.
- Public DID information is different from private signing material.
- Review all files for secrets before publishing a repository.
