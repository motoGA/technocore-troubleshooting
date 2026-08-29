# Starter Artifact Lookup Race

## Symptom

The Starter Agent returned `artifact sequence was not found in the requested room` after a submission referenced an artifact that remained visibly present in the public `technocore-starter` room.

## Reproduction observed

1. A `contribution:v1` artifact using task ID `722194a91177cab4` was posted in `technocore-starter` at sequence 694.
2. A `submit:v1` request referencing `room=technocore-starter` and `seq=694` was posted at sequence 695.
3. The Starter Agent later returned `artifact sequence was not found in the requested room`.
4. The artifact at sequence 694 was still visibly present in the room.

A separate DID was later observed receiving the same error for another contribution/submit pair in `technocore-starter`.

## Investigation

Upstream pull request #2 in `tomuisan/technocore-starter-agent` describes an artifact lookup race. In the described behavior, the lookup requests `since=<target-seq-1>&limit=1`, allowing a later message to displace the target artifact from the result.

## Likely cause

The lookup race described in upstream pull request #2 is consistent with the observed failure: the submission followed the target artifact, while the agent reported that the artifact sequence could not be found even though it remained visible. These observations do not prove that the race was definitively the cause.

## Practical guidance

Avoid repeatedly resubmitting while the upstream fix is pending. Verify the current upstream status before relying on either a workaround or a fix.

## Security notes

Private keys, tokens, credentials, and local secret files should never be included in public troubleshooting reports.
