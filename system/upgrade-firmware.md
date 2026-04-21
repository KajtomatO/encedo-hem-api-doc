# Firmware Upgrade Endpoints

PPA-only, three-step firmware upgrade: **upload → check → install**.

- **Upload scope:** strictly `system:upgrade` (narrower than below — `system:config` is not accepted on upload).
- **Check/install scopes:** `system:upgrade` or `system:config`.
- **Allowed `sub`:** any (including `E` tokens from external authenticator flows, if they hold the right scope).
- On an **uninitialised** device the entire flow is available without a Bearer token (first-boot provisioning).
- A firmware install while another install is in progress returns `409`.

---

## POST /api/system/upgrade/upload_fw

Upload a new firmware image to `/tmp/<filename>` on the internal filesystem. Requires `USB_MSC_AVAILABLE` (PPA only).

### Request

```
POST /api/system/upgrade/upload_fw
Authorization: Bearer <token>
Content-Disposition: attachment; filename="firmware.bin"
Content-Type: application/octet-stream
Expect: 100-continue
```

- Filename must end in `.bin` or `.hex`. `.bin` sets `Session.NewFWtype=1`, `.hex` sets `=2`. Anything else → `406`.
- `Content-Length` must be `< 3 MiB`. Larger → `400`.
- Absolute cap on any file-upload POST is 16 MiB → `413`.
- Body is streamed to `/tmp/<filename>`. On success `Session.NewFW` is set to `1` (= uploaded, not yet checked).

### Response

`200 OK` on success (empty body). No JSON.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Body too large (>3 MiB), or filename check failed. |
| 401 | Device initialised and no Bearer. |
| 403 | Scope not `system:upgrade` (audit-logged `LOG_TYPE_FAILED_SCOPE_CHECK`). |
| 406 | Filename didn't end in `.bin` / `.hex`. |
| 413 | >16 MiB. |

---

## GET /api/system/upgrade/check_fw

Asynchronously validate the uploaded firmware (signature via `checkFirmwareBINFile` / `checkFirmwareHexFile`). Non-blocking: the first call starts a FreeRTOS task and returns `201`; poll until `200` (valid) or `406` (invalid).

### Request

```
GET /api/system/upgrade/check_fw
Authorization: Bearer <token>
```

### Response (empty body; status is the signal)

| Session state | Returned status |
|---------------|-----------------|
| No file uploaded yet (`NewFW=0`) | `404` |
| File uploaded, task not yet started | `201` (task just started by this call) |
| Check task running | `202` |
| Check passed (`NewFW=2`) | `200` |
| Check failed (`NewFW=3`) | `406` |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope not `system:upgrade` or `system:config`. |
| 404 | No uploaded firmware to check. |

---

## GET /api/system/upgrade/install_fw

Commit the validated firmware: reboots into bootloader-upgrade mode, which flashes the new image. Caller's connection is closed immediately; the device applies the upgrade after a 2-second delay.

### Request

```
GET /api/system/upgrade/install_fw
Authorization: Bearer <token>
```

**Preconditions:**
- `Session.NewFW == 2` (check has passed). Otherwise → `406`.
- No other software install in progress (otherwise `409`; set by `software_install_busy_setifclr`).

### Response

`200 OK` with `Connection: close`. Audit-logged `LOG_TYPE_UPGRADE` (OK). Device then calls `restart_bootloader(BOOT_GOTO_UPGRADE)`.

On next boot, `GET /api/system/status` will include `"fw_upgrade": true` on the first read.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope not `system:upgrade` or `system:config`. |
| 406 | `Session.NewFW != 2`. Audit-logged `LOG_TYPE_UPGRADE` (ERROR). |
| 409 | Other install in progress. Audit-logged `LOG_TYPE_UPGRADE` (ERROR). |

## Encedo Manager Usage

```javascript
// build.js:976 — upload (XHR; supports retry on chunk failure)
xhr.open('POST', self.encedo_url + '/api/system/upgrade/upload_fw', true);
// build.js:1111 — poll check_fw
xhr.open('GET', self.encedo_url + '/api/system/upgrade/check_fw');
// build.js:1075 — install_fw
xhr.open('GET', self.encedo_url + '/api/system/upgrade/install_fw');
```

The Manager downloads `api.encedo.com/download/firmware/{version}/bin` and relays it to the device.

Manager references: [encedo.js:855](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L855), [build.js:976](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L976)

## Test Suite Usage

```php
// test_7.php:184 (upload)
http_transaction("https", "POST", $cfg_domain, "/api/system/upgrade/upload_fw", $ret_val, $fw_data, $token);
// test_7.php:191 (check)
http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/check_fw", $ret_val, $post_dummy, $token);
// test_7.php:203 (install)
http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/install_fw", $ret_val, $post_dummy, $token);
```

Test references: [test_7.php:184](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L184), [test_7.php:191](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L191), [test_7.php:203](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L203)
