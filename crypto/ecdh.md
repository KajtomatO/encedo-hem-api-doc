# POST /api/crypto/ecdh

Perform an Elliptic Curve Diffie-Hellman key exchange, returning either raw or hashed result.

## Request

```
POST /api/crypto/ecdh
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}#{descr}`

```json
{
  "kid": "32char_hex_key_id",
  "pubkey": "base64_external_public_key",
  "ext_kid": "32char_hex_external_key_id",
  "alg": "SHA2-256"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) |
| `pubkey` | String | No | Base64-encoded external public key |
| `ext_kid` | String | No | External key ID (32-char hex) |
| `alg` | String | No | Hash algorithm (omit for raw ECDH) |

Either `pubkey` or `ext_kid` must be provided. The key type of `ext_kid` or `pubkey` must match the `kid` key type.

### Supported Algorithms (for hashed output)

SHA2-256, SHA2-384, SHA2-512, SHA3-256, SHA3-384, SHA3-512

## Response

**Status:** `200 OK`

```json
{
  "ecdh": "base64_encoded_ecdh_result"
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

Not registered in the `_endpoints` map and not directly called from the Manager UI. Documented in official docs only.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:571
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/ecdh", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:571](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L571), [test_11.php:579](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L579)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** This endpoint (`/api/crypto/ecdh`) is documented in the official docs but is **not present** in [Encedo Manager](https://github.com/encedo/encedo-manager)'s `_endpoints` registry or `scopes.js`. It is not called anywhere in the Manager JavaScript. The related key derivation is handled via `/api/keymgmt/derive` instead, which has its own `keymgmt:ecdh` scope.
