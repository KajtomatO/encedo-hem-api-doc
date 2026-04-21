# GET /api/system/upgrade/usbmode

Reboot the device into USB serial (CDC ACM) upgrade mode. In this mode the device exposes a serial port (`/dev/ttyACM0` on Linux) to the bootloader, which accepts an Intel HEX firmware stream. Available on both PPA and EPA builds.

## Request

```
GET /api/system/upgrade/usbmode
Authorization: Bearer <token>
```

**Authentication:**
- On an **initialised** device: Bearer with scope `system:upgrade` or `system:config`.
- On a **fresh/uninitialised** device: no auth.
- Another install in progress → `409`.

## Response

**Status:** `200 OK` with `Connection: close`, followed after a 2-second delay by `restart_bootloader(BOOT_GOTO_USB)`. The device then reboots into the bootloader's USB serial mode; the HTTP listener is gone until the next power-cycle or bootloader exit.

No JSON body.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope not `system:upgrade` or `system:config`. |
| 406 | Generic operation failure. Audit-logged `LOG_TYPE_UPGRADE` (ERROR). |
| 409 | Another install in progress. |

## Notes

- The bootloader running after this call is a separate component; documenting its serial protocol is out of scope for this API reference.
- This endpoint is the only path that triggers `BOOT_GOTO_USB` — the web upload flow uses `BOOT_GOTO_UPGRADE` instead.

## Encedo Manager Usage

Not exercised by the current Manager build.

## Test Suite Usage

```php
// test_7.php:90
http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/usbmode", $ret_val, $post_dummy, $token);
```

Test references: [test_7.php:90](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L90), [upgrade2diag.php:76](https://github.com/encedo/hem-api-tester/blob/main/upgrade2diag.php#L76)
