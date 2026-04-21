# POST /api/crypto/cipher/encrypt

Encrypt data with AES-ECB, AES-CBC, or AES-GCM. The encryption key is either an AES key in the repo (**direct**) or an ECDH-derived AES key (**ECDH mode**, same HKDF scheme as [`cipher/wrap`](cipher-wrap.md)).

The IV is **always generated server-side** — callers do not supply one. For CBC and GCM the IV is echoed back in the response. PKCS#7 padding is applied automatically for CBC.

## Request

```
POST /api/crypto/cipher/encrypt
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
  "msg":     "<base64_plaintext>",
  "alg":     "AES256-GCM",
  "aad":     "<base64_aad>",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>",
  "ctx":     "<base64_hkdf_info>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. AES key or ECDH-capable private key. |
| `msg` | String | Yes | Base64 plaintext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. For `-ECB` the decoded length must be a multiple of `AES_BLOCK_SIZE` (16); for `-CBC` padding is added automatically. |
| `alg` | String | Yes | Exactly 10 characters: `AES128-ECB`, `AES192-ECB`, `AES256-ECB`, `AES128-CBC`, `AES192-CBC`, `AES256-CBC`, `AES128-GCM`, `AES192-GCM`, `AES256-GCM`. |
| `aad` | String | No | Base64 additional authenticated data (GCM only, ≤ `AES_BLOCK_SIZE` = 16 bytes). |
| `ext_kid` | String | ECDH only | Peer public key already in repo. |
| `pubkey` | String | ECDH only | Base64 raw peer public key (≤ 66 bytes). |
| `ctx` | String | ECDH only | Base64 HKDF `info` (≤ 64 bytes). Default `"encedo"`. |

For ECDH mode, exactly one of `ext_kid` / `pubkey` must be present.

`iv` is **not** a request field — the firmware generates a fresh random 16-byte IV on every call.

## Response

**Status:** `200 OK`

```json
{
  "ciphertext": "<base64>",
  "iv":         "<base64_iv>",
  "tag":        "<base64_gcm_tag>"
}
```

| Field | When | Notes |
|-------|------|-------|
| `ciphertext` | Always | Base64 ciphertext. |
| `iv` | CBC and GCM | Base64 of the generated 16-byte IV. Omitted for ECB. |
| `tag` | GCM only | Base64 of the 16-byte GCM auth tag. |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing fields, `alg` length ≠ 10 or unknown mode/strength, or ECB input not block-aligned. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Key type mismatch, ECDH failure, or cipher failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_ENCRYPT`. Plaintext buffer is zeroed after the call.

## Notes

- Default `alg` shape in the parser is `AES256-GCM` but **`alg` is effectively required** — any value with length ≠ 10 returns 400. Send the exact string.
- Use GCM when you need authenticated encryption. CBC has no integrity — pair with [`/api/crypto/hmac/hash`](hmac-hash.md) if you need it.

## Encedo Manager Usage

Registered in endpoints/scopes map ([build.js:9278](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9278), [scopes.js:815](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L815)); not called directly by Manager UI.

## Test Suite Usage

```php
// test_11.php:352
http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/encrypt", $ret_val, $post_array, $token);
```

Test references: [test_11.php:352](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L352), [test_11.php:373](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L373), [test_11.php:397](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L397), [test_11.php:627](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L627), [test_11.php:650](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L650)
