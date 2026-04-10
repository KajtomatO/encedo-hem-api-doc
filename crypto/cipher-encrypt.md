# POST /api/crypto/cipher/encrypt

Encrypt data using AES.

## Request

```
POST /api/crypto/cipher/encrypt
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_plaintext",
  "alg": "AES256-GCM",
  "iv": "base64_initialization_vector",
  "aad": "base64_additional_authenticated_data"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID of AES key (32-char hex) |
| `msg` | String | Yes | Base64-encoded plaintext data |
| `alg` | String | Yes | AES algorithm variant (see supported list) |
| `iv` | String | No | Base64-encoded initialization vector (required for CBC and GCM modes) |
| `aad` | String | No | Base64-encoded additional authenticated data (GCM mode only) |

### Supported Algorithms

| Algorithm | Mode | Notes |
|-----------|------|-------|
| AES128-ECB | ECB | No IV needed |
| AES192-ECB | ECB | No IV needed |
| AES256-ECB | ECB | No IV needed |
| AES128-CBC | CBC | IV required |
| AES192-CBC | CBC | IV required |
| AES256-CBC | CBC | IV required |
| AES128-GCM | GCM | IV required, supports AAD |
| AES192-GCM | GCM | IV required, supports AAD |
| AES256-GCM | GCM | IV required, supports AAD |

## Response

**Status:** `200 OK`

```json
{
  "ciphertext": "base64_encoded_ciphertext",
  "iv": "base64_initialization_vector",
  "tag": "base64_authentication_tag"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ciphertext` | String | Base64-encoded encrypted data |
| `iv` | String | Base64-encoded IV (returned for CBC/GCM modes) |
| `tag` | String | Base64-encoded authentication tag (GCM mode only) |

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

Registered in `_endpoints` ([build.js:9278](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9278), [scopes.js:815](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L815)) but not directly called in the Manager UI. Request/response schemas confirmed via [HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_11.php](https://github.com/encedo/hem-api-tester/blob/main/test_11.php)).

## Test Suite Usage

```php
// hem-api-tester/test_11.php:352
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/encrypt", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:352](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L352), [test_11.php:373](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L373), [test_11.php:397](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L397), [test_11.php:627](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L627), [test_11.php:650](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L650)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** The official documentation page returns **404**. Full request/response schemas were reverse-engineered from the [HEM API test suite](https://github.com/encedo/hem-api-tester), not the official docs.
