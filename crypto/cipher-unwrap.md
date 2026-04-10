# POST /api/crypto/cipher/unwrap

Unwrap (decrypt) secret data using the NIST Key Wrapping scheme (AES).

## Request

```
POST /api/crypto/cipher/unwrap
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "alg": "AES256",
  "msg": "base64_encoded_wrapped_data"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID of AES key (32-char hex) |
| `alg` | String | Yes | Algorithm: AES128, AES192, or AES256 |
| `msg` | String | Yes | Base64-encoded wrapped data from wrap operation |

## Response

**Status:** `200 OK`

```json
{
  "unwrapped": "base64_encoded_unwrapped_data"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `unwrapped` | String | Base64-encoded unwrapped (decrypted) key material |

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

Registered in `_endpoints` ([build.js:9293](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9293), [scopes.js:830](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L830)) but not directly called in the Manager UI. Schemas confirmed via [HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_11.php](https://github.com/encedo/hem-api-tester/blob/main/test_11.php)).

## Test Suite Usage

```php
// hem-api-tester/test_11.php:477
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/unwrap", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:477](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L477), [test_11.php:679](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L679), [test_11.php:699](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L699)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** Official documentation page returns **404**. Schemas from test suite.
