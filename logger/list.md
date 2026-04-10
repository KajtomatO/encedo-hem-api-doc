# GET /api/logger/list/:offset

Get a list of audit log file IDs.

## Request

```
GET /api/logger/list/:offset
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `logger:get`

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `offset` | Number | Yes | Number of log entries to skip |

## Response

**Status:** `200 OK`

```json
{
  "total": 3,
  "id": ["62310b2b", "62310bf6", "6231f603"]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `total` | Number | Total number of log entries |
| `id` | Array | Array of log file IDs (hex strings) |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 404 | No log entries found |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:5199, core2.js:2937
app.api('api/logger/list' + offset).then(function(data) {
    // data.total, data.id[]
});
```

**Source references:** [build.js:5199](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5199), [core2.js:2937](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2937)

## Test Suite Usage

```php
// hem-api-tester/test_9.php:79
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/logger/list/0", $ret_val, "", $token);

// hem-api-tester/test_9.php:111
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/logger/list/0", $ret_val, "", $token);
```

**Source references:** [test_9.php:79](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L79), [test_9.php:111](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L111)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
