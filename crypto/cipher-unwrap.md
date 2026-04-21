# POST /api/crypto/cipher/unwrap

Inverse of [`/api/crypto/cipher/wrap`](cipher-wrap.md). Unwrap an AES-wrapped blob using either a stored AES KEK (direct) or an ECDH-derived AES KEK (ECDH mode).

## Request

```
POST /api/crypto/cipher/unwrap
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
  "msg":     "<base64_wrapped_blob>",
  "alg":     "AES256",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>",
  "ctx":     "<base64_hkdf_info>",
  "iv":      "<base64_iv>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. AES KEK (direct) or ECDH-capable private key (ECDH mode). |
| `msg` | String | Yes | Base64 wrapped ciphertext. Decoded length must be a multiple of `KEYWRAP_BLOCK_SIZE` (8). |
| `alg` | String | No | `AES128` / `AES192` / `AES256`. Default `AES256`. |
| `ext_kid` | String | ECDH only | Peer public key already in repo. |
| `pubkey` | String | ECDH only | Base64 raw peer public key (≤ 66 bytes). |
| `ctx` | String | ECDH only | Base64 HKDF `info` (≤ 64 bytes). Default `"encedo"`. Must match the value used by wrap. |
| `iv` | String | No | Base64 IV, exactly 8 bytes. Must match the wrap-side IV. |

For the ECDH flow, exactly one of `ext_kid` / `pubkey` must be present.

## Response

**Status:** `200 OK`

```json
{ "unwrapped": "<base64_plaintext>" }
```

Plaintext is `ciphertext_len - 8` bytes before base64.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `msg`, bad `alg`, or ciphertext length not a multiple of 8. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | KEK not suitable, ECDH failure, or integrity check failure (wrong key/IV/ctx). |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_UNWRAP`.

## Encedo Manager Usage

Registered in endpoints/scopes map ([build.js:9293](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9293), [scopes.js:830](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L830)); not called directly by Manager UI.

## Test Suite Usage

```php
// test_11.php:477
http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/unwrap", $ret_val, $post_array, $token);
```

Test references: [test_11.php:477](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L477), [test_11.php:679](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L679), [test_11.php:699](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L699)
