# GET /api/system/shutdown

Shut down the device and enter a secure state.

## Request

```
GET /api/system/shutdown
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `system:shutdown` (primary), or `system:config` (alternative)

**Allowed users:** User, Master, ExtAuth

## Response

**Status:** `200 OK` -- Device shuts down

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 409 | Incorrect internal state |

## Encedo Manager Usage

Not directly invoked from the [Encedo Manager](https://github.com/encedo/encedo-manager) UI, but the scope `system:shutdown` is defined in [scopes.js:89](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L89).

**Source references:** [scopes.js:89](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L89) (scope definition)

## Test Suite Usage

```php
// hem-api-tester/test_8.php:219
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/shutdown", $ret_val, $post_dummy, $token);
```

**Source references:** [test_8.php:219](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L219)

## Official Docs vs Encedo Manager

**Status:** Documented.

> **DISCREPANCY:** The official docs list this endpoint. [Encedo Manager](https://github.com/encedo/encedo-manager) defines the scope `system:shutdown` in `scopes.js` but there is **no direct API call** to `/api/system/shutdown` in the JavaScript. The scope consent dialog exists, suggesting this endpoint is used by third-party apps or triggered through a different flow not visible in the Manager code.
