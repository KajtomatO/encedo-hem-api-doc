# GET /api/system/reboot

Reboot the device. All session state (including every issued Bearer token, since tokens are HMAC'd under the per-boot session key) is invalidated.

## Request

```
GET /api/system/reboot
Authorization: Bearer <token>
```

**Authentication:**
- On an **initialised** device: Bearer required, scope must be one of:
  - `system:upgrade`
  - `system:config`
  - `system:shutdown`
- On a **fresh/uninitialised** device: no auth (convenience during first-boot provisioning).
- If a firmware/UI install is in progress → `409`.

## Response

**Status:** `200 OK` is returned **immediately**, with `Connection: close`, and then the device waits 2 seconds and calls `restart_bootloader(BOOT_NORMAL)`.

No body.

### Side effects

- Audit log entry `LOG_TYPE_REBOOT` with result `OK` (caller's `sub` included).
- Callers should drop any cached tokens and re-authenticate after the device comes back.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device is initialised and Bearer missing/invalid. |
| 403 | Scope mismatch (logged as `LOG_TYPE_FAILED_SCOPE_CHECK`). |
| 409 | Firmware/UI install in progress. |

## Encedo Manager Usage

```javascript
app.api('api/system/reboot');
```

Manager references: [build.js:5180](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5180)

## Test Suite Usage

```php
http_transaction("https", "GET", $cfg_domain, "/api/system/reboot", $ret_val, $post_dummy, $token);
```

Test references: [test_1.php:166](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L166)
