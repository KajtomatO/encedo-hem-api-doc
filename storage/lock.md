# GET /api/storage/lock

Lock all embedded disks. Encedo PPA only.

## Request

```
GET /api/storage/lock
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `storage:disk`

**Availability:** Encedo PPA only

## Response

**Status:** `200 OK`

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:1839, build.js:7848-7879, core2.js:5556-5587
app.api('api/storage/lock').then(function(data) {
    app.api('api/system/status').then(function(data) {
        // Verify storage locked via status
    });
});
```

**Source references:** [build.js:1839](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1839), [build.js:7848](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7848), [build.js:7879](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7879), [core2.js:5556](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5556), [core2.js:5587](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5587)

## Test Suite Usage

```php
// hem-api-tester/test_12.php:69
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/storage/lock", $ret_val, "", $token);

// hem-api-tester/test_12.php:118
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/storage/lock", $ret_val, "", $token);
```

**Source references:** [test_12.php:69](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L69), [test_12.php:118](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L118)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
