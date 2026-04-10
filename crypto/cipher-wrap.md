# POST /api/crypto/cipher/wrap

Wrap (encrypt) secret data using the NIST Key Wrapping scheme (AES). If no data is provided, generates random secret data and wraps it.

## Request

```
POST /api/crypto/cipher/wrap
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "alg": "AES256",
  "msg": "base64_encoded_secret_data"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID of AES key (32-char hex) |
| `alg` | String | Yes | Algorithm: AES128, AES192, or AES256 |
| `msg` | String | No | Base64-encoded data to wrap (generated if omitted) |

## Response

**Status:** `200 OK`

```json
{
  "wrapped": "base64_encoded_wrapped_data"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `wrapped` | String | Base64-encoded wrapped (encrypted) key material |

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

Registered in `_endpoints` ([build.js:9288](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9288), [scopes.js:825](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L825)) but not directly called in the Manager UI. Schemas confirmed via [HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_11.php](https://github.com/encedo/hem-api-tester/blob/main/test_11.php)).

## Test Suite Usage

```php
// hem-api-tester/test_11.php:469
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/wrap", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:469](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L469), [test_11.php:671](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L671), [test_11.php:691](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L691)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** Official documentation page returns **404**. Schemas from test suite.
