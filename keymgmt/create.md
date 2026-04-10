# POST /api/keymgmt/create

Create a new cryptographic key and store it in the device's secure repository.

## Request

```
POST /api/keymgmt/create
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:gen`

**Allowed users:** User, ExtAuth

```json
{
  "label": "My signing key",
  "type": "ED25519",
  "mode": "sign",
  "descr": "base64_encoded_description"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | String | Yes | Human-readable key identifier |
| `type` | String | Yes | Cryptographic key type (see supported types) |
| `mode` | String | No | Operation mode (NIST ECC only) |
| `descr` | String | No | Base64-encoded supplementary description |

### Supported Key Types

**ECC:** SECP256R1, SECP384R1, SECP521R1, SECP256K1, CURVE25519, CURVE448, ED25519, ED448

**HMAC:** SHA2-256, SHA2-384, SHA2-512, SHA3-256, SHA3-384, SHA3-512

**AES:** AES128, AES192, AES256

**Post-Quantum:** MLKEM512, MLKEM768, MLKEM1024, MLDSA44, MLDSA65, MLDSA87

## Response

**Status:** `200 OK`

```json
{
  "kid": "09bd0958e1499ecfd51ea62a3f49a84c"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `kid` | String | 32-character hex key identifier |

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
// build.js:6662, core2.js:4367
app.api('api/keymgmt/create', 'POST', newKeyData).then(function(data) {
    // data.kid contains the new key ID
});
```

**Source references:** [build.js:6662](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6662), [core2.js:4367](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4367)

## Test Suite Usage

```php
// hem-api-tester/test_10.php:115
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/create", $ret_val, $keycreate_arg, $token);
```

**Source references:** [test_10.php:115](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L115)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **KEY TYPE NAMING (from test suite):** AES key types use no dash (`AES128`, `AES192`, `AES256`), not `AES-128`. HMAC key types use hash name only (`SHA2-256`, `SHA3-512`), not `HMAC-SHA2-256`. Confirmed via [HEM API test suite](https://github.com/encedo/hem-api-tester) `test_10.php`.
