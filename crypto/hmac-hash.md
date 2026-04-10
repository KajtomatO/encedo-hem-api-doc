# POST /api/crypto/hmac/hash

Generate an HMAC hash of the given data.

## Request

```
POST /api/crypto/hmac/hash
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}` (key-specific usage scope)

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_message",
  "alg": "SHA2-256",
  "ext_kid": "32char_hex_external_key_id",
  "pubkey": "base64_external_public_key"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) |
| `msg` | String | Yes | Base64-encoded message (max 2048 bytes) |
| `alg` | String | No | Algorithm: SHA2-256, SHA2-384, SHA2-512, SHA3-256, SHA3-384, SHA3-512 |
| `ext_kid` | String | No | External key ID |
| `pubkey` | String | No | Base64-encoded external public key |

## Response

**Status:** `200 OK`

```json
{
  "mac": "base64_encoded_hmac"
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

Registered in `_endpoints` ([build.js:9253](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9253)) but **not directly called** in the Manager UI. Available for third-party app integration.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:88
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/hmac/hash", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:88](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L88), [test_11.php:593](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L593), [test_11.php:733](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L733)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
