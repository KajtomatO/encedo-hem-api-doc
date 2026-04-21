# POST /api/crypto/exdsa/sign

Produce a digital signature over a message using an ECDSA (NIST-P / secp256k1) or EdDSA (Ed25519 / Ed448) private key.

## Request

```
POST /api/crypto/exdsa/sign
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
  "kid": "<32_char_hex_kid>",
  "msg": "<base64_message>",
  "alg": "Ed25519",
  "ctx": "<base64_context>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of an ECDSA or EdDSA signing key. |
| `msg` | String | Yes | Base64 plaintext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. |
| `alg` | String | Yes | Digest/variant selector — see below. Validated as a ≤24-byte label. |
| `ctx` | String | No | Base64 context (Ed25519ctx / Ed448 / Ed448ph). Decoded length ≤ 255 (RFC 8032 cap). |

### Supported algorithms

| `alg` | Applies to key types |
|-------|----------------------|
| `SHA256WithECDSA` | `SECP256R1`, `SECP256K1` |
| `SHA384WithECDSA` | `SECP384R1` |
| `SHA512WithECDSA` | `SECP521R1` |
| `Ed25519` | `ED25519` (pure) |
| `Ed25519ph` | `ED25519` (pre-hashed SHA-512) |
| `Ed25519ctx` | `ED25519` with context |
| `Ed448` | `ED448` |
| `Ed448ph` | `ED448` (pre-hashed SHAKE256) |

The firmware rejects any other string at parse time.

## Response

**Status:** `200 OK`

```json
{ "sign": "<base64_signature>" }
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing fields, invalid `alg` label, or base64 decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Signing failed (wrong key type for `alg`, or low-level crypto failure). |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_EXDSA_SIGN`.

## Encedo Manager Usage

Registered in endpoints/scopes map; not called directly by Manager UI ([build.js:9263](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9263)).

## Test Suite Usage

```php
// test_11.php:186 (+ seven more call sites covering all algorithms)
http_transaction("https", "POST", $cfg_domain, "/api/crypto/exdsa/sign", $ret_val, $post_array, $token);
```

Test references: [test_11.php:186](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L186), [test_11.php:200](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L200), [test_11.php:213](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L213), [test_11.php:230](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L230), [test_11.php:244](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L244), [test_11.php:259](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L259), [test_11.php:272](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L272), [test_11.php:285](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L285)
