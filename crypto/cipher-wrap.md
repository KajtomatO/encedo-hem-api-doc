# POST /api/crypto/cipher/wrap

Wrap arbitrary key material with AES key wrap (RFC 3394 / RFC 5649, depending on length). The wrapping key is either:

- an AES key already in the repo (**direct** mode), or
- an AES key **derived on the fly** from `ECDH(kid_private, peer_public) + HKDF(info=ctx ?? "encedo")` (**ECDH** mode).

Output is one cohesive ciphertext: `base64(wrapped)`.

## Request

```
POST /api/crypto/cipher/wrap
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
  "alg":     "AES256",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>",
  "ctx":     "<base64_hkdf_info>",
  "iv":      "<base64_iv>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. AES key (direct) **or** ECDH-capable private key (ECDH mode). |
| `msg` | String | Yes | Base64 of the plaintext to wrap. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. Must be non-empty — firmware rejects missing / empty input with `400`. |
| `alg` | String | No | `AES128` / `AES192` / `AES256`. Default `AES256`. Must match the KEK's strength. |
| `ext_kid` | String | ECDH only | Peer public key already in the repo. |
| `pubkey` | String | ECDH only | Base64 raw peer public key (≤ 66 bytes). |
| `ctx` | String | ECDH only | Base64 HKDF `info` (≤ 64 bytes). Default `"encedo"`. |
| `iv` | String | No | Base64 IV, exactly `KEYWRAP_BLOCK_SIZE` (8) bytes. Default: RFC 3394 constant. |

For the ECDH flow, exactly one of `ext_kid` / `pubkey` must be present; omit both for direct wrap.

## Response

**Status:** `200 OK`

```json
{ "wrapped": "<base64_wrapped_output>" }
```

Ciphertext is `plaintext_len + 8` bytes before base64.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `msg`, bad `alg`, unknown field shape, or decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | KEK not suitable, ECDH failure, or wrap failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_WRAP`. Plaintext buffer is zeroed after the call.

## Notes

- If you want the device to **generate** the material rather than wrap caller-supplied bytes, see [`/api/keymgmt/create`](../keymgmt/create.md). The wrap endpoint always needs `msg`.
- In ECDH mode the wrapping key never leaves the device — only the wrapped blob is returned.

## Encedo Manager Usage

Registered in endpoints/scopes map ([build.js:9288](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9288), [scopes.js:825](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L825)); not called directly by Manager UI.

## Test Suite Usage

```php
// test_11.php:469
http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/wrap", $ret_val, $post_array, $token);
```

Test references: [test_11.php:469](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L469), [test_11.php:671](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L671), [test_11.php:691](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L691)
