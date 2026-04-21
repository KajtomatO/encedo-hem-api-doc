# POST /api/crypto/hmac/hash

Produce a MAC over a message using a key from the repository. The key can be an HMAC key (direct hash), or an ECDH-capable private key combined with a peer (via `ext_kid` or `pubkey`) — in the ECDH variant the firmware derives a per-message symmetric key via ECDH + HKDF before HMACing.

## Request

```
POST /api/crypto/hmac/hash
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** **exact** match `keymgmt:use:<kid_hex>` — the JWT has to name the same `kid` that appears in the body.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Body

```json
{
  "kid":     "<32_char_hex_kid>",
  "msg":     "<base64_message>",
  "alg":     "SHA2-256",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of an HMAC key **or** an ECDH-capable private key. |
| `msg` | String | Yes | Base64 plaintext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. |
| `alg` | String | If ECDH | `SHA2-256`, `SHA2-384`, `SHA2-512`, `SHA3-256`, `SHA3-384`, `SHA3-512`. For HMAC keys the hash is implied by the key type; for ECDH keys `alg` selects both the HKDF hash and the HMAC hash. |
| `ext_kid` | String | ECDH only | 32-char hex of a peer public key already in the repo. |
| `pubkey` | String | ECDH only | Base64 raw peer public key (max 66 bytes, fits P-521). |

For the ECDH flow, exactly one of `ext_kid` / `pubkey` is used.

## Response

**Status:** `200 OK`

```json
{ "mac": "<base64_mac>" }
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing fields, invalid `alg`, or base64 decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope is not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Key type wrong for requested flow, ECDH failure, or HMAC failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_HMAC_HASH`.

## Notes

- The `keymgmt:use:<kid>` scope is minted via [`/api/auth/ext/request`](../auth/ext-request.md). Tokens with this scope are also valid for all other `/api/crypto/*` endpoints for the same `kid`.
- Paired-authenticator tokens arrive with a `#<metadata>` suffix on the scope — that suffix is stripped by [`/api/auth/ext/request`](../auth/ext-request.md) before the exact-match check, so it does not affect usability here.

## Encedo Manager Usage

Registered in the endpoints/scopes map for third-party integration; not directly called by Manager UI ([build.js:9253](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9253)).

## Test Suite Usage

```php
// test_11.php:88
http_transaction("https", "POST", $cfg_domain, "/api/crypto/hmac/hash", $ret_val, $post_array, $token);
```

Test references: [test_11.php:88](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L88), [test_11.php:593](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L593), [test_11.php:733](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L733)
