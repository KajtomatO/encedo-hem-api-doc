# GET /api/logger/key

Get the audit log signing key and a signed nonce for log verification.

## Request

```
GET /api/logger/key
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `logger:get`

## Response

**Status:** `200 OK`

```json
{
  "key": "base64_signing_public_key",
  "nonce": "base64_random_nonce",
  "nonce_signed": "base64_signed_nonce"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `key` | String | Audit log signing public key (base64) |
| `nonce` | String | Random nonce (base64) |
| `nonce_signed` | String | Nonce signed with the log signing key (base64) |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:5217, build.js:5540, build.js:5606
// core2.js:2955, core2.js:3245, core2.js:3311
app.api('api/logger/key').then(function(signature) {
    // Used to verify log file integrity
    // signature.key, signature.nonce, signature.nonce_signed
});
```

**Source references:** [build.js:5217](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5217), [build.js:5540](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5540), [build.js:5606](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5606), [core2.js:2955](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2955), [core2.js:3245](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3245), [core2.js:3311](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3311)

## Test Suite Usage

```php
// hem-api-tester/test_3.php:174
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/logger/key", $ret_val, "", $token);

// hem-api-tester/test_9.php:62
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/logger/key", $ret_val, "", $token);
```

**Source references:** [test_3.php:174](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L174), [test_9.php:62](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L62)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
