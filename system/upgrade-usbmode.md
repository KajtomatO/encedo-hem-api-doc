# GET /api/system/upgrade/usbmode

Enable USB serial (CDC ACM) mode for firmware upgrade via direct USB connection. After calling this endpoint, the device exposes a serial port (e.g., `/dev/ttyACM0` on Linux) and accepts firmware in Intel HEX format.

## Request

```
GET /api/system/upgrade/usbmode
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `system:upgrade`

## Response

**Status:** `200 OK`

After a successful response, the device enters USB serial mode. Firmware can be sent over the serial port (e.g., using `sx` / XMODEM or a terminal tool on Windows).

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

**Not present in the Manager UI.** This endpoint is not registered in `_endpoints` and not called in [Encedo Manager](https://github.com/encedo/encedo-manager).

Discovered via the [HEM API test suite](https://github.com/encedo/hem-api-tester) (`test_7.php:90`, `upgrade2diag.php:76`), where it is used before sending firmware over the serial port.

## Test Suite Usage

```php
// hem-api-tester/test_7.php:90
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/usbmode", $ret_val, $post_dummy, $token);
```

**Source references:** [test_7.php:90](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L90), [upgrade2diag.php:76](https://github.com/encedo/hem-api-tester/blob/main/upgrade2diag.php#L76)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** This endpoint is **not documented** in the official docs and **not present** in [Encedo Manager](https://github.com/encedo/encedo-manager). It exists only in the [HEM API test suite](https://github.com/encedo/hem-api-tester), used for USB-based firmware upgrades as an alternative to the web upload flow.
