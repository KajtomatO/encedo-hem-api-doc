# POST /api/crypto/exdsa/verify

Verify an ECDSA / EdDSA signature. Mirror of [`/api/crypto/exdsa/sign`](exdsa-sign.md) — same algorithm vocabulary, same scope model.

## Request

```
POST /api/crypto/exdsa/verify
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
  "kid":  "<32_char_hex_kid>",
  "msg":  "<base64_message>",
  "sign": "<base64_signature>",
  "alg":  "Ed25519",
  "ctx":  "<base64_context>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of the signing key (public half used for verify). |
| `msg` | String | Yes | Base64 plaintext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. |
| `sign` | String | Yes | Base64 signature. Decoded length ≤ 2·66+16 (fits largest ECDSA/EdDSA sig). |
| `alg` | String | Yes | Same set as sign: `SHA256WithECDSA`, `SHA384WithECDSA`, `SHA512WithECDSA`, `Ed25519`, `Ed25519ph`, `Ed25519ctx`, `Ed448`, `Ed448ph`. |
| `ctx` | String | No | Base64 context (Ed25519ctx / Ed448 / Ed448ph). Decoded length ≤ 255. |

## Response

**Status:** `200 OK` — no body on successful verification.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing fields, invalid `alg`, or base64 decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Signature verification failed, or wrong key type for `alg`. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_EXDSA_VERIFY`.

## Encedo Manager Usage

Registered in endpoints/scopes map; not called directly by Manager UI ([build.js:9268](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9268)).

> **Manager map quirk:** the endpoint appears twice in both `_endpoints` ([build.js:9268](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9268) and [build.js:9273](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9273)) and `scopes.js` ([scopes.js:805](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L805) and [:810](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L810)). JS object key collision means last one wins; harmless but worth noting.

## Test Suite Usage

```php
// test_11.php:192 (+ seven more call sites covering all algorithms)
http_transaction("https", "POST", $cfg_domain, "/api/crypto/exdsa/verify", $ret_val, $post_array, $token);
```

Test references: [test_11.php:192](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L192), [test_11.php:206](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L206), [test_11.php:219](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L219), [test_11.php:236](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L236), [test_11.php:250](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L250), [test_11.php:265](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L265), [test_11.php:278](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L278), [test_11.php:291](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L291)
