# POST /api/crypto/exdsa/verify

Verify a digital signature (ECDSA or EdDSA).

## Request

```
POST /api/crypto/exdsa/verify
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_message",
  "sign": "base64_signature_to_verify",
  "alg": "Ed25519",
  "ctx": "base64_context_data"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) |
| `msg` | String | Yes | Base64-encoded message (max 2048 bytes) |
| `sign` | String | Yes | Signature to validate |
| `alg` | String | Yes | Algorithm (same list as sign endpoint) |
| `ctx` | String | No | Base64-encoded context data |

## Response

**Status:** `200 OK` -- Signature verified

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed (verification failed) |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

Registered in `_endpoints` ([build.js:9268](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9268)) but not directly called in the Manager UI.

> **NOTE:** This endpoint is duplicated in both `_endpoints` ([build.js:9268](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9268) and [build.js:9273](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9273)) and `scopes.js` ([scopes.js:805](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L805) and [scopes.js:810](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L810)) -- identical entries appear twice. This is a JS object key collision (last one wins).

## Test Suite Usage

```php
// hem-api-tester/test_11.php:192
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/exdsa/verify", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:192](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L192), [test_11.php:206](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L206), [test_11.php:219](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L219), [test_11.php:236](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L236), [test_11.php:250](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L250), [test_11.php:265](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L265), [test_11.php:278](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L278), [test_11.php:291](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L291)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
