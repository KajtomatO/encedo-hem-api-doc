# POST /api/keymgmt/import

Import a public key into the key repository as a trusted key.

## Request

```
POST /api/keymgmt/import
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:imp`

**Allowed users:** User, ExtAuth

```json
{
  "label": "Partner's key",
  "pubkey": "base64_encoded_public_key",
  "type": "ED25519",
  "mode": "sign",
  "descr": "base64_description"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | String | Yes | Key identifier |
| `pubkey` | String | Yes | Base64-encoded public key material |
| `type` | String | Yes | Key algorithm type |
| `mode` | String | No | Operation mode (NIST ECC only) |
| `descr` | String | No | Base64-encoded description |

### Supported Key Types

SECP256R1, SECP384R1, SECP521R1, SECP256K1, CURVE25519, CURVE448, ED25519, ED448, MLKEM512, MLKEM768, MLKEM1024, MLDSA44, MLDSA65, MLDSA87

## Response

**Status:** `200 OK`

```json
{
  "kid": "d51ea62a3f49a84cadbd0958e1499ecf"
}
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Invalid argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Insufficient access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

```javascript
// build.js:6751, core2.js:4456
app.api('api/keymgmt/import', 'POST', newKeyData);
```

**Source references:** [build.js:6751](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6751), [core2.js:4456](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4456)

## Test Suite Usage

```php
// hem-api-tester/test_8.php:293
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/import", $ret_val, $keyimport_arg, $token);

// hem-api-tester/test_10.php:276
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/import", $ret_val, $keyimport_arg, $token);
```

**Source references:** [test_8.php:293](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L293), [test_10.php:276](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L276)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
