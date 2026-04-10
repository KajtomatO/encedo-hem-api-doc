# GET /api/storage/unlock/:mode

Unlock an embedded disk for reading or writing. Encedo PPA only.

The disk number and access mode are determined by the JWT token's scope, not by URL parameters. However, [Encedo Manager](https://github.com/encedo/encedo-manager) uses `/ro` and `/rw` suffixes to indicate the desired access mode.

## Request

```
GET /api/storage/unlock/ro
GET /api/storage/unlock/rw
Authorization: Bearer <JWT_TOKEN>
```

**Scopes:**
- `storage:disk0` -- Unlock primary (regular) storage, read-only
- `storage:disk0:rw` -- Unlock primary storage, read-write
- `storage:disk1` -- Unlock hidden (encrypted) storage, read-only
- `storage:disk1:rw` -- Unlock hidden storage, read-write

**Availability:** Encedo PPA only (EPA returns 404)

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
// build.js:7729-7817, core2.js:5437-5525
// Determine endpoint based on user's write permission choice
var endpoint = 'api/storage/unlock/ro';
if (writePermission) {
    endpoint = 'api/storage/unlock/rw';
}
app.api(endpoint).then(function(data) {
    // Check status after unlock
    app.api('api/system/status').then(function(data) {
        // Verify storage state changed
    });
});
```

**Source references:** [build.js:1813](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1813), [build.js:7729-7817](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7729), [core2.js:5437-5525](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5437)

## Test Suite Usage

```php
// hem-api-tester/test_12.php:77
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/storage/unlock", $ret_val, "", $token);

// hem-api-tester/test_12.php:87
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/storage/unlock", $ret_val, "", $token);
```

**Source references:** [test_12.php:77](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L77), [test_12.php:87](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L87)

## Official Docs vs Encedo Manager

> **DISCREPANCY (URL path):** The official docs show the endpoint as simply `/api/storage/unlock` and state that disk number and mode are derived from the JWT scope. [Encedo Manager](https://github.com/encedo/encedo-manager) uses `/api/storage/unlock/ro` and `/api/storage/unlock/rw` suffixes. The older `encedo.js:1691` uses the plain `/api/storage/unlock` without suffix, while the newer `build.js` and `core2.js` consistently use `/ro` or `/rw` suffixes. The `/ro` and `/rw` path variants are **not documented** in the official docs.

> **DISCREPANCY (endpoint registry):** The `_endpoints` map registers `/api/storage/unlock` four times (for each disk/mode combination), all as `GET` with no path suffixes. The actual code uses path suffixes `/ro` and `/rw`.

> **TEST SUITE NOTE:** The [HEM API test suite](https://github.com/encedo/hem-api-tester) (`test_4.php`) uses the plain `/api/storage/unlock` path (no `/ro` or `/rw` suffix), aligning with the official docs and the endpoint registry -- not [Encedo Manager](https://github.com/encedo/encedo-manager). This suggests the `/ro` and `/rw` suffixes are a Manager-specific convention, and the device accepts the plain path.
