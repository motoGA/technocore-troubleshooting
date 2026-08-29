# DID Note Sharded Path

## Symptom

A DID directory-note lookup initially failed when using a legacy, non-sharded path form. This observation does not establish that every legacy path always fails.

## Fingerprint calculation

Calculate the fingerprint from the exact DID string. It is the first 16 lowercase hexadecimal characters of the DID string's SHA-256 hash.

```powershell
$didValue = '<DID_STRING>'
$sha256 = [System.Security.Cryptography.SHA256]::Create()

try {
    $didBytes = [System.Text.Encoding]::UTF8.GetBytes($didValue)
    $hashBytes = $sha256.ComputeHash($didBytes)
    $hashHex = -join ($hashBytes | ForEach-Object { $_.ToString('x2') })
    $fingerprint = $hashHex.Substring(0, 16)
} finally {
    $sha256.Dispose()
}

$fingerprint
```

## Sharded path format

The directory-note path splits the 16-character fingerprint after its first two characters:

```text
/kv/did-XX/YYYYYYYYYYYYYY
```

Here, `XX` is the first 2 characters of the fingerprint and `YYYYYYYYYYYYYY` is the remaining 14 characters. The path can be constructed in PowerShell after calculating `$fingerprint`:

```powershell
$shardedPath = '/kv/did-{0}/{1}' -f $fingerprint.Substring(0, 2), $fingerprint.Substring(2, 14)
$shardedPath
```

## Verification

After the fingerprint was calculated correctly and the sharded path was used, the DID note was successfully retrieved. The retrieved note contained the DID itself.

## Common mistakes

- Using the legacy, non-sharded path form for this lookup.
- Guessing the fingerprint or manually transcribing the path instead of calculating it from the exact DID string.
- Failing to use the required 16-character lowercase hexadecimal fingerprint or splitting it somewhere other than after the first 2 characters.

## Security notes

A DID is public identity material. Private signing keys, seeds, tokens, credentials, and local secret-file contents must never be published.
