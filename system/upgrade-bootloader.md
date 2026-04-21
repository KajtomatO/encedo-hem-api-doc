# Bootloader Upgrade Endpoints

Bootloader-replacement flow. **Compiled into the firmware only when `DIAG_MODE_ON` is defined** — production PPA/EPA builds do **not** expose these endpoints. Also requires `USB_MSC_AVAILABLE` (PPA-only code path).

When DIAG mode is off:
- `POST /api/system/upgrade/upload_bootldr` → `404` (no dispatch match).
- `GET /api/system/upgrade/install_bl` → `404`.

There is **no** `check_bl` endpoint; the install path reuses the uploaded image as-is.

---

## POST /api/system/upgrade/upload_bootldr

Upload a new bootloader image. The firmware uses the hardcoded filename `bootloader` and the storage target literal `flash` (passed to `api_post_system_upgrade_upload_ext`); the `Content-Disposition` filename from the request is ignored for this endpoint.

### Request

```
POST /api/system/upgrade/upload_bootldr
Authorization: Bearer <token>
Content-Type: application/octet-stream
Expect: 100-continue
```

- Upload scope: `system:upgrade` (on initialised devices). Pre-init devices skip the check.
- Absolute cap 16 MiB (shared with other upload endpoints) → `413`.

### Response

`200 OK` on successful upload. No JSON body.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope not `system:upgrade`. |
| 404 | DIAG mode not compiled in. |
| 413 | >16 MiB. |

---

## GET /api/system/upgrade/install_bl

DIAG-only bootloader install. Unlocks the bootloader flash region, erases three sectors at `MEM_FLASH_ADDRRESS_BOOTLOADER`, then copies `MEM_FLASH_SECTOR_SIZE` bytes from `MEM_FLASH_ADDRRESS_RESERVED` into the bootloader region, and verifies by hashing the written region with SHA-1.

### Request

```
GET /api/system/upgrade/install_bl
Authorization: Bearer <token>
```

- Scope: `system:upgrade` or `system:config`.

### Response

```json
{ "hash": "<sha1_hex>" }
```

Status `200` on success; `400` on verification failure (sector not fully erased before the write).

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Flash erase/verify failed. |
| 401 | Device initialised and no Bearer. |
| 403 | Scope mismatch. |
| 404 | DIAG mode not compiled in. |

## Notes

- The firmware comment on this handler reads `"TBD — validate signature (errors), validate if publisher is ok (origin)"` — **the current implementation does not verify the signature/publisher of the new bootloader before flashing**. Tracked in [FIRMWARE_NOTES.md](../FIRMWARE_NOTES.md).
- There is no reboot after install; callers must follow up with `GET /api/system/reboot`.

## Encedo Manager Usage

Defined in the endpoint registry but **not directly called** by current Manager builds. An `update_bootloader_page` HTML stub exists without corresponding JS logic.

Manager references: [build.js:9142](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9142), [scopes.js:679](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L679)

## Test Suite Usage

Not exercised by the current `hem-api-tester` suite.
