# POST /api/crypto/hmac/verify

Verify an HMAC hash of the given data.

## Request

```
POST /api/crypto/hmac/verify
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_message",
  "mac": "base64_mac_to_verify",
  "alg": "SHA2-256",
  "ext_kid": "32char_hex_external_key_id",
  "pubkey": "base64_external_public_key"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) |
| `msg` | String | Yes | Base64-encoded message (max 2048 bytes) |
| `mac` | String | Yes | MAC value from hash operation to verify |
| `alg` | String | No | Algorithm selection |
| `ext_kid` | String | No | External key ID |
| `pubkey` | String | No | Base64-encoded external public key |

## Response

**Status:** `200 OK` -- HMAC verified successfully

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

Registered in `_endpoints` ([build.js:9258](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9258)) but not directly called in the Manager UI.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:104
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/hmac/verify", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:104](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L104), [test_11.php:601](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L601)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
