# POST /api/keymgmt/derive

Derive a new key using ECDH and store it in the secure repository.

## Request

```
POST /api/keymgmt/derive
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:ecdh` (primary), or `keymgmt:gen` (alternative)

**Allowed users:** User, ExtAuth

```json
{
  "kid": "32char_hex_key_id",
  "label": "Derived key",
  "type": "CURVE25519",
  "descr": "base64_description",
  "ext_kid": "32char_hex_external_key_id",
  "mode": "sign",
  "pubkey": "base64_external_public_key"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Source key ID (32-char hex) |
| `label` | String | Yes | Label for the derived key |
| `type` | String | Yes | Type of key to create |
| `descr` | String | No | Base64-encoded description |
| `ext_kid` | String | No | External key ID (32-char hex) |
| `mode` | String | No | Key operation mode (NIST ECC only) |
| `pubkey` | String | No | Base64-encoded external public key |

## Response

**Status:** `200 OK`

```json
{
  "kid": "bd0958e1499ecfd51ea62a3f49a84cad"
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

Not directly invoked in the Manager UI, but registered in the endpoints map ([build.js:9233](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9233)).

**Source references:** [build.js:9233](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9233) (endpoint definition), [scopes.js:770](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L770) (scope definition)

## Test Suite Usage

```php
// hem-api-tester/test_10.php:175
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/derive", $ret_val, $keyderive_arg, $token);
```

**Source references:** [test_10.php:175](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L175)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **DISCREPANCY:** The `_endpoints` map in [build.js:9233](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9233) and [scopes.js:770](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L770) both register this as `/api/keymgmt/derive`. However, the official docs URL path is also `/api/keymgmt/derive`. While the endpoint is registered, **no direct call to this endpoint exists** in the Manager UI JavaScript. It appears to be available only for third-party apps.
