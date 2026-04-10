# POST /api/system/upgrade/upload_bootldr

Upload a new bootloader upgrade file.

**Scope:** `system:upgrade`

## Request

```
POST /api/system/upgrade/upload_bootldr
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/octet-stream
```

**Body:** Raw binary bootloader file

## Response

**Status:** `200 OK`

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |

## Encedo Manager Usage

Defined in the endpoint registry ([build.js:9142](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9142), [scopes.js:679](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L679)) but **not directly called** in [Encedo Manager](https://github.com/encedo/encedo-manager). The UI has an `update_bootloader_page` (`index.html:407`) placeholder but the upload logic is not implemented in the current JavaScript.

**Source references:** [build.js:9142](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9142) (endpoint definition), [scopes.js:679](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L679) (scope definition)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** This endpoint is registered in the `_endpoints` map and has a scope definition, but there is **no corresponding `check_bootldr` or `install_bootldr` endpoint** defined anywhere in [Encedo Manager](https://github.com/encedo/encedo-manager) or official docs. The bootloader update flow appears incomplete compared to firmware and UI flows, which both have upload/check/install triads. The HTML page `update_bootloader_page` exists but the JS logic is absent.
