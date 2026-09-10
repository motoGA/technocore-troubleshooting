# Technocore KV: verify ambiguous 5xx writes with read-after-write

## Context

Observed while running an end-to-end FLOP Labs tclk/1 PaperRail deal against the shared Technocore service on 2026-09-02. PaperRail held no real value.

## Observed behavior

### Case 1: HTTP 524, but the write committed

- The KV SET request returned HTTP 524.
- An immediate GET of the same namespace and key returned the exact expected value.
- Treating every 5xx response as a failed write would have caused an unnecessary retry.

### Case 2: HTTP 503, and the write did not commit

- A separate KV SET request returned HTTP 503.
- An immediate GET of the same namespace and key returned HTTP 404.
- In this case the SET had actually failed.

## Recommended client behavior

For a KV SET that returns HTTP 5xx:

1. GET the same namespace and key.
2. Compare the returned value with the exact value the client attempted to write.
3. If the values match, treat the operation as committed.
4. If the key is absent, the value differs, or verification is unavailable, preserve the original failure and apply the retry policy appropriate to the write condition.

A 5xx response alone proves neither success nor failure. Blind retrying can duplicate work or overwrite a newer value. Conditional-write semantics and HTTP 409 handling must remain separate from this check.

## Minimal pseudocode

```text
response = SET(namespace, key, expected_value, condition)
if response.status is 5xx:
    observed = GET(namespace, key)
    if observed == expected_value:
        return committed
    return ambiguous_or_failed
return normal_status_handling(response)
```

## Evidence references

- tclk public offer seq: 472
- tclk contract: 0xced9c25d92bd21a874eda67df07953f808775a0ad2914ec0b3ddab42b0901510
- Technocore tclk evidence anchor: technocore-starter seq 5023
- Starter contribution artifact: technocore-starter seq 6343
