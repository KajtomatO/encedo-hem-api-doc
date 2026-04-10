# POST /api/crypto/exdsa/sign

Digitally sign data using ECDSA or EdDSA.

## Request

```
POST /api/crypto/exdsa/sign
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_message",
  "alg": "Ed25519",
  "ctx": "base64_context_data"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) |
| `msg` | String | Yes | Base64-encoded message (max 2048 bytes) |
| `alg` | String | Yes | Algorithm (see supported list) |
| `ctx` | String | No | Base64-encoded context data |

### Supported Algorithms

SHA256WithECDSA, SHA384WithECDSA, SHA512WithECDSA, Ed25519, Ed25519ph, Ed25519ctx, Ed448, Ed448ph

## Response

**Status:** `200 OK`

```json
{
  "sign": "base64_encoded_signature"
}
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

Registered in `_endpoints` ([build.js:9263](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9263)) but not directly called in the Manager UI.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:186
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/exdsa/sign", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:186](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L186), [test_11.php:200](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L200), [test_11.php:213](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L213), [test_11.php:230](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L230), [test_11.php:244](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L244), [test_11.php:259](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L259), [test_11.php:272](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L272), [test_11.php:285](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L285)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
