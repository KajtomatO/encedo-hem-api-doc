# Encedo HEM REST API Documentation

> **Source of truth:** This documentation was reverse-engineered from the [Encedo Manager](https://github.com/encedo/encedo-manager) source code and the [HEM API test suite](https://github.com/encedo/hem-api-tester), then cross-referenced with the [official Encedo HEM API docs](https://docs.encedo.com/hem-api/reference/api-reference). Where the official docs are incomplete or contradict the implementations, discrepancies are noted.

## Base URL

All local device endpoints use the device's URL as base (e.g., `https://my.ence.do`).
Cloud endpoints use `https://api.encedo.com`.

## Authentication

Most endpoints require a JWT Bearer token in the `Authorization` header:
```
Authorization: Bearer <JWT_TOKEN>
```

Tokens are obtained via `/api/auth/token` (local password auth) or `/api/auth/ext/token` (mobile auth). Each token is scoped to specific operations.

**Unauthenticated endpoints:** `/api/system/version`, `/api/system/status`, `/api/system/checkin`, `/api/auth/token`, `/api/auth/ext/token`, `/api/auth/ext/request`, `/api/auth/init`.

## Common Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Process started (upgrade validation) |
| 202 | Process ongoing (upgrade validation) |
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope / RTC not set |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Endpoint Index

### System
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/system/version` | _none_ | [version.md](system/version.md) |
| GET | `/api/system/status` | _none_ | [status.md](system/status.md) |
| GET | `/api/system/checkin` | _none_ | [checkin.md](system/checkin.md) |
| POST | `/api/system/checkin` | _none_ | [checkin.md](system/checkin.md) |
| GET | `/api/system/config` | `system:config` | [config.md](system/config.md) |
| POST | `/api/system/config` | `system:config` | [config.md](system/config.md) |
| GET | `/api/system/config/attestation` | _optional_ | [config-attestation.md](system/config-attestation.md) |
| POST | `/api/system/config/provisioning` | _none_ | [config-provisioning.md](system/config-provisioning.md) |
| GET | `/api/system/selftest` | _any valid token_ | [selftest.md](system/selftest.md) |
| GET | `/api/system/reboot` | `system:upgrade` | [reboot.md](system/reboot.md) |
| GET | `/api/system/shutdown` | `system:shutdown` | [shutdown.md](system/shutdown.md) |
| POST | `/api/system/upgrade/upload_fw` | `system:upgrade` | [upgrade-firmware.md](system/upgrade-firmware.md) |
| GET | `/api/system/upgrade/check_fw` | `system:upgrade` | [upgrade-firmware.md](system/upgrade-firmware.md) |
| GET | `/api/system/upgrade/install_fw` | `system:upgrade` | [upgrade-firmware.md](system/upgrade-firmware.md) |
| POST | `/api/system/upgrade/upload_ui` | `system:upgrade` | [upgrade-ui.md](system/upgrade-ui.md) |
| GET | `/api/system/upgrade/check_ui` | `system:upgrade` | [upgrade-ui.md](system/upgrade-ui.md) |
| GET | `/api/system/upgrade/install_ui` | `system:upgrade` | [upgrade-ui.md](system/upgrade-ui.md) |
| POST | `/api/system/upgrade/upload_bootldr` | `system:upgrade` | [upgrade-bootloader.md](system/upgrade-bootloader.md) |
| GET | `/api/system/upgrade/usbmode` | `system:upgrade` | [upgrade-usbmode.md](system/upgrade-usbmode.md) |

### Diagnostics (diagnostic firmware only)
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/diag/*` (7 endpoints) | _varies_ | [diag.md](system/diag.md) |

### Authorization
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/auth/init` | _none_ | [auth-init.md](auth/init.md) |
| POST | `/api/auth/init` | _none_ | [auth-init.md](auth/init.md) |
| GET | `/api/auth/token` | _none_ | [auth-token.md](auth/token.md) |
| POST | `/api/auth/token` | _none_ | [auth-token.md](auth/token.md) |
| POST | `/api/auth/ext/init` | `auth:ext:pair` | [ext-init.md](auth/ext-init.md) |
| POST | `/api/auth/ext/validate` | `auth:ext:pair` | [ext-validate.md](auth/ext-validate.md) |
| POST | `/api/auth/ext/mac` | `auth:ext:pair` | [ext-mac.md](auth/ext-mac.md) |
| POST | `/api/auth/ext/request` | _none_ | [ext-request.md](auth/ext-request.md) |
| POST | `/api/auth/ext/token` | _none_ | [ext-token.md](auth/ext-token.md) |

### Key Management
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| POST | `/api/keymgmt/create` | `keymgmt:gen` | [create.md](keymgmt/create.md) |
| POST | `/api/keymgmt/derive` | `keymgmt:ecdh` | [derive.md](keymgmt/derive.md) |
| POST | `/api/keymgmt/import` | `keymgmt:imp` | [import.md](keymgmt/import.md) |
| POST | `/api/keymgmt/update` | `keymgmt:upd` | [update.md](keymgmt/update.md) |
| DELETE | `/api/keymgmt/delete/:kid` | `keymgmt:del` | [delete.md](keymgmt/delete.md) |
| GET | `/api/keymgmt/get/:kid` | `keymgmt:get` | [get.md](keymgmt/get.md) |
| GET | `/api/keymgmt/list/:offset/:limit` | `keymgmt:list` | [list.md](keymgmt/list.md) |
| POST | `/api/keymgmt/search` | `keymgmt:search` | [search.md](keymgmt/search.md) |

### Storage (PPA only)
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/storage/unlock/ro` | `storage:disk0` or `storage:disk1` | [unlock.md](storage/unlock.md) |
| GET | `/api/storage/unlock/rw` | `storage:disk0:rw` or `storage:disk1:rw` | [unlock.md](storage/unlock.md) |
| GET | `/api/storage/lock` | `storage:disk` | [lock.md](storage/lock.md) |

### Audit Log
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/logger/key` | `logger:get` | [key.md](logger/key.md) |
| GET | `/api/logger/list/:offset` | `logger:get` | [list.md](logger/list.md) |
| GET | `/api/logger/:id` | `logger:get` | [get.md](logger/get.md) |
| DELETE | `/api/logger/:id` | `logger:del` | [delete.md](logger/delete.md) |

### Cryptography Operations
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| POST | `/api/crypto/hmac/hash` | `keymgmt:use:*` | [hmac-hash.md](crypto/hmac-hash.md) |
| POST | `/api/crypto/hmac/verify` | `keymgmt:use:*` | [hmac-verify.md](crypto/hmac-verify.md) |
| POST | `/api/crypto/exdsa/sign` | `keymgmt:use:*` | [exdsa-sign.md](crypto/exdsa-sign.md) |
| POST | `/api/crypto/exdsa/verify` | `keymgmt:use:*` | [exdsa-verify.md](crypto/exdsa-verify.md) |
| POST | `/api/crypto/ecdh` | `keymgmt:use:*` | [ecdh.md](crypto/ecdh.md) |
| POST | `/api/crypto/cipher/encrypt` | `keymgmt:use:*` | [cipher-encrypt.md](crypto/cipher-encrypt.md) |
| POST | `/api/crypto/cipher/decrypt` | `keymgmt:use:*` | [cipher-decrypt.md](crypto/cipher-decrypt.md) |
| POST | `/api/crypto/cipher/wrap` | `keymgmt:use:*` | [cipher-wrap.md](crypto/cipher-wrap.md) |
| POST | `/api/crypto/cipher/unwrap` | `keymgmt:use:*` | [cipher-unwrap.md](crypto/cipher-unwrap.md) |

### Post-Quantum Cryptography
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| POST | `/api/crypto/pqc/mldsa/sign` | `keymgmt:use:*` | [pqc/mldsa-sign.md](crypto/pqc/mldsa-sign.md) |
| POST | `/api/crypto/pqc/mldsa/verify` | `keymgmt:use:*` | [pqc/mldsa-verify.md](crypto/pqc/mldsa-verify.md) |
| POST | `/api/crypto/pqc/mlkem/encaps` | `keymgmt:use:*` | [pqc/mlkem-encaps.md](crypto/pqc/mlkem-encaps.md) |
| POST | `/api/crypto/pqc/mlkem/decaps` | `keymgmt:use:*` | [pqc/mlkem-decaps.md](crypto/pqc/mlkem-decaps.md) |

---

## Documentation vs Reality: Discrepancy Summary

See [DISCREPANCIES.md](DISCREPANCIES.md) for the full list of differences between the official Encedo documentation and the actual implementations.
