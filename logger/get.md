# GET /api/logger/{id}

Download a single audit log file as plain text. **PPA only** (compiled behind `USB_MSC_AVAILABLE`; EPA returns `404`).

## Request

```
GET /api/logger/{id}
Authorization: Bearer <token>
```

**Authentication:** Bearer required. Scope must start with `logger:get`.

**Preconditions:** Initialised — otherwise `409`.

### Path parameters

| Parameter | Description |
|-----------|-------------|
| `id` | Hex-encoded log file id, as returned by [`/api/logger/list`](list.md) (e.g. `62310b2b`). |

The handler rejects `list` and `list/…` paths at a prior branch — those go to the list endpoint.

## Response

**Status:** `200 OK`
**Content-Type:** `text/plain`
**Transfer:** Content-Length set to the full file size, then the file body streamed in 1 KiB chunks (keep-alive).

The body is the raw log file as stored on the microSD — one JSON-like record per line, each record including an Ed25519 signature (verify with [`/api/logger/key`](key.md)) and an HMAC-SHA256 chain hash linking to the previous entry.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope. |
| 404 | File not found. |
| 406 | File locked by another operation (`FR_LOCKED`). |
| 409 | Not initialised. |
| 500 | Allocation failure. |

Audit log event on success/failure: `LOG_TYPE_LOGGER_READ_LOG`.

## Notes

- An I/O error mid-stream truncates the response — the TCP connection is closed with `Connection: close` and the caller sees a short read vs. the declared `Content-Length`. The audit record is written with `LOG_RESULT_FAILED` in that case.
- Verifying a log file end-to-end requires the current `Session.EIDkeySign` public key from `/api/logger/key` **plus** an audit trail of that same public key (e.g. recorded at pairing time). A fresh reboot preserves the key; a wipeout rotates it and old logs become unverifiable against the new key.

## Encedo Manager Usage

```javascript
// build.js:5542 — Blob download mode
app.api('api/logger/' + id, 'GET_FILE');

// build.js:5610 — inline mode
app.api('api/logger/' + id).then(data => { /* parse records */ });
```

Manager references: [build.js:5540](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5540), [core2.js:3245](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3245)

## Test Suite Usage

```php
// test_9.php:118
http_transaction("https", "GET", $cfg_domain, "/api/logger/" . $log_id, $ret_val, "", $token);
```

Test references: [test_9.php:118](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L118)
