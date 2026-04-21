# POST /api/crypto/cipher/decrypt

Inverse of [`/api/crypto/cipher/encrypt`](cipher-encrypt.md). Decrypt AES-ECB / AES-CBC / AES-GCM ciphertext using a stored AES key or an ECDH-derived AES key.

PKCS#7 padding is removed automatically for CBC. GCM tag is verified — tag mismatch returns `406`.

## Request

```
POST /api/crypto/cipher/decrypt
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
  "msg":     "<base64_ciphertext>",
  "alg":     "AES256-GCM",
  "iv":      "<base64_iv>",
  "tag":     "<base64_gcm_tag>",
  "aad":     "<base64_aad>",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>",
  "ctx":     "<base64_hkdf_info>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. AES key or ECDH-capable private key. |
| `msg` | String | Yes | Base64 ciphertext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN + AES_BLOCK_SIZE`. For ECB must be a multiple of 16. |
| `alg` | String | Yes | Exactly 10 chars, same vocabulary as encrypt. |
| `iv` | String | CBC/GCM | Base64, exactly 16 bytes. |
| `tag` | String | GCM | Base64, exactly 16 bytes. |
| `aad` | String | GCM only | Base64, ≤ 16 bytes. Must match the value used at encrypt time. |
| `ext_kid` | String | ECDH only | Peer public key in repo. |
| `pubkey` | String | ECDH only | Base64 raw peer public key (≤ 66 bytes). |
| `ctx` | String | ECDH only | Base64 HKDF `info` (≤ 64 bytes). Default `"encedo"`. |

For ECDH mode, exactly one of `ext_kid` / `pubkey` must be present.

## Response

**Status:** `200 OK`

```json
{ "plaintext": "<base64_plaintext>" }
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing required fields, bad `alg`, IV / tag wrong length, ECB ciphertext not block-aligned. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Cipher failure (wrong key, GCM tag mismatch, ECDH failure). |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_DECRYPT`. Ciphertext buffer is zeroed after the call.

## Notes

- CBC padding removal includes a fixup for the DEV-367 bug where the previous padding-strip logic mishandled plaintexts whose final byte equalled a valid padding value. Current firmware strips cleanly for all cases where `1 ≤ pad ≤ 16`.

## Encedo Manager Usage

Registered in endpoints/scopes map ([build.js:9283](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9283), [scopes.js:820](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L820)); not called directly by Manager UI.

## Test Suite Usage

```php
// test_11.php:360
http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/decrypt", $ret_val, $post_array, $token);
```

Test references: [test_11.php:360](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L360), [test_11.php:383](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L383), [test_11.php:409](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L409), [test_11.php:637](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L637), [test_11.php:660](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L660)
