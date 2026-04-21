# GET /api/logger/list[/{offset}]

List audit log file IDs stored on the microSD. **PPA only** (compiled behind `USB_MSC_AVAILABLE`; EPA returns `404`).

## Request

```
GET /api/logger/list
GET /api/logger/list/0        # offset 0 — newest page
GET /api/logger/list/20       # skip first 20 entries
Authorization: Bearer <token>
```

**Authentication:** Bearer required. Scope must start with `logger:get`.

**Preconditions:** Initialised — otherwise `409`.

### Path parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `offset` | Integer | Optional — number of newest files to skip. Omitted / invalid → treated as `0`. |

## Response

**Status:** `200 OK`

```json
{
  "total": 3,
  "id": ["62310b2b", "62310bf6", "6231f603"]
}
```

| Field | Description |
|-------|-------------|
| `total` | Total number of audit log files on the device. |
| `id` | Page of log file IDs (hex-encoded filenames) starting at `offset`. Page size is set by the firmware's log listing implementation. |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope. |
| 404 | Logger listing failed (directory read error or no logs). |
| 406 | Malformed `offset` (non-numeric suffix). |
| 409 | Not initialised. |

Audit log event: `LOG_TYPE_LOGGER_LIST_LOG` on completion (both success and failure).

## Notes

- File IDs are derived from creation timestamps — sort lexicographically is chronological.
- Each file contains one or more signed, chain-HMAC'd audit log entries. Use [`/api/logger/{id}`](get.md) to download and [`/api/logger/key`](key.md) to verify signatures.

## Encedo Manager Usage

```javascript
// build.js:5199, core2.js:2937
app.api('api/logger/list/' + offset).then(function(data) {
    // data.total, data.id[]
});
```

Manager references: [build.js:5199](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5199), [core2.js:2937](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2937)

## Test Suite Usage

```php
// test_9.php:79
http_transaction("https", "GET", $cfg_domain, "/api/logger/list/0", $ret_val, "", $token);
```

Test references: [test_9.php:79](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L79), [test_9.php:111](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L111)
