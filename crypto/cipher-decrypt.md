# POST /api/crypto/cipher/decrypt

Decrypt data using AES.

## Request

```
POST /api/crypto/cipher/decrypt
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_ciphertext",
  "alg": "AES256-GCM",
  "iv": "base64_initialization_vector",
  "tag": "base64_authentication_tag"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID of AES key (32-char hex) |
| `msg` | String | Yes | Base64-encoded ciphertext |
| `alg` | String | Yes | AES algorithm variant (same as encrypt) |
| `iv` | String | No | Base64-encoded IV (required for CBC and GCM modes) |
| `tag` | String | No | Base64-encoded authentication tag (required for GCM mode) |

### Supported Algorithms

Same as [cipher-encrypt.md](cipher-encrypt.md): AES128/192/256-ECB, AES128/192/256-CBC, AES128/192/256-GCM.

## Response

**Status:** `200 OK`

```json
{
  "plaintext": "base64_encoded_plaintext"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `plaintext` | String | Base64-encoded decrypted data |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token / GCM tag verification failed |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

Registered in `_endpoints` ([build.js:9283](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9283), [scopes.js:820](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L820)) but not directly called in the Manager UI. Schemas confirmed via [HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_11.php](https://github.com/encedo/hem-api-tester/blob/main/test_11.php)).

## Test Suite Usage

```php
// hem-api-tester/test_11.php:360
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/cipher/decrypt", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:360](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L360), [test_11.php:383](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L383), [test_11.php:409](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L409), [test_11.php:637](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L637), [test_11.php:660](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L660)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** Official documentation page returns **404**. Schemas from test suite.
