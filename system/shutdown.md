# GET /api/system/shutdown

Stop the web-server tasks and the USB device stack. The device stops responding on the network until power-cycled.

## Request

```
GET /api/system/shutdown
Authorization: Bearer <token>
```

**Authentication:** Bearer required **always** (no pre-init shortcut, unlike `reboot` / `upgrade`). Scope must be one of:
- `system:config`
- `system:shutdown`

If a firmware/UI install is in progress → `409`.

## Response

**Status:** `200 OK` is returned **immediately**, with `Connection: close`, then the device waits 2 seconds, calls `udc_stop()` (kills USB), and deletes the web-server task. From that point the device is unreachable via this interface.

No body.

### Side effects

- Audit log entry `LOG_TYPE_SHUTDOWN` with result `OK` (caller's `sub` included).

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Bearer missing/invalid. |
| 403 | Scope mismatch (logged as `LOG_TYPE_FAILED_SCOPE_CHECK`). |
| 409 | Firmware/UI install in progress. |

## Notes

- Unlike `reboot`, this handler does **not** skip auth on uninitialised devices — even a fresh device refuses `401` without a token (though no fresh device has issued any tokens, so shutdown is effectively unavailable pre-init).
- There is no "wake up" endpoint; recovery from shutdown requires a physical power cycle.

## Encedo Manager Usage

```javascript
app.api('api/system/shutdown');
```

Manager references: [build.js:5192](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5192)

## Test Suite Usage

```php
http_transaction("https", "GET", $cfg_domain, "/api/system/shutdown", $ret_val, $post_dummy, $token);
```

Test references: [test_1.php:176](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L176)
