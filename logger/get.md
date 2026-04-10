# GET /api/logger/:id

Download a specific audit log file.

## Request

```
GET /api/logger/:id
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `logger:get`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | String | Log file ID (hex string, e.g., "62310b2b") |

## Response

**Status:** `200 OK`

**Content-Type:** `text/plain` (log file content)

The response is plain text log data. [Encedo Manager](https://github.com/encedo/encedo-manager) supports two modes:
- Normal: JSON-parsed response
- File download: Raw text saved as `logger_file.txt` via Blob download

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 404 | File not found |
| 406 | Operation failed (e.g., file busy) |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:5542 -- File download mode
app.api('api/logger/' + id, 'GET_FILE').then(function(data) {
    // Triggers blob download as 'logger_file.txt'
});

// build.js:5610 -- JSON parse mode
app.api('api/logger/' + id).then(function(data) {
    // Returns parsed log data
});
```

**Source references:** [build.js:5540-5610](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5540), [core2.js:3245-3315](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3245)

## Test Suite Usage

```php
// hem-api-tester/test_9.php:118
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/logger/" . $log_id, $ret_val, "", $token);
```

**Source references:** [test_9.php:118](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L118)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
