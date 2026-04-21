# POST /api/crypto/hmac/verify

Verify an HMAC against a message. Mirror of [`/api/crypto/hmac/hash`](hmac-hash.md) — same key flows (HMAC direct or ECDH-derived), same scope rules.

## Request

```
POST /api/crypto/hmac/verify
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** **exact** match `keymgmt:use:<kid_hex>`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Body

```json
{
  "kid":     "<32_char_hex_kid>",
  "msg":     "<base64_message>",
  "mac":     "<base64_mac_to_verify>",
  "alg":     "SHA2-256",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. HMAC or ECDH-capable key. |
| `msg` | String | Yes | Base64. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. |
| `mac` | String | Yes | Base64 of the MAC to check. Decoded length ≤ `WC_SHA512_DIGEST_SIZE` (64). |
| `alg` | String | If ECDH | `SHA2-256`/`384`/`512`, `SHA3-256`/`384`/`512`. |
| `ext_kid` | String | ECDH only | Peer public key already in repo. |
| `pubkey` | String | ECDH only | Base64 raw peer public key (≤ 66 bytes). |

## Response

**Status:** `200 OK` — no body on successful verification.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing fields, invalid `alg`, or base64 decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | MAC mismatch, ECDH failure, or key type wrong for requested flow. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_HMAC_HASH` (both hash and verify share the same event type in the current firmware — filter by `LOG_RESULT_OK/FAILED` to distinguish pass/fail, not by operation).

## Encedo Manager Usage

Registered in endpoints/scopes map; not called directly by Manager UI ([build.js:9258](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9258)).

## Test Suite Usage

```php
// test_11.php:104
http_transaction("https", "POST", $cfg_domain, "/api/crypto/hmac/verify", $ret_val, $post_array, $token);
```

Test references: [test_11.php:104](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L104), [test_11.php:601](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L601)
