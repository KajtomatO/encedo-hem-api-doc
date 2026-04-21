# Encedo HEM REST API Documentation

This repository is a community-maintained reference for the REST API exposed by Encedo HEM devices (PPA and EPA variants). It is organised for third-party implementers who need to talk to the device directly.

## Sources

Each endpoint page reflects the actual device behaviour — request validation, scope checks, failure modes, audit events. Where a behaviour is hard to observe from outside, we cross-reference two public artifacts:

- [Encedo Manager](https://github.com/encedo/encedo-manager) — the official web client. Referenced for usage patterns and scope vocabulary.
- [HEM API test suite](https://github.com/encedo/hem-api-tester) — the PHP test harness. Referenced for request/response shapes, especially where the public [Encedo docs](https://docs.encedo.com/hem-api/reference/api-reference) are missing or out of date.

Notes on firmware-level oddities — bugs, undocumented behaviours, disabled endpoints — live in [FIRMWARE_NOTES.md](FIRMWARE_NOTES.md). Notes on where the external artifacts disagree with each other or with the device live under [discrepancies/](discrepancies/).

## Base URL

All device endpoints are served from the device's own address (e.g. `https://my.ence.do`). Cloud endpoints use `https://api.encedo.com`.

## Authentication

Most endpoints require a JWT Bearer token:

```
Authorization: Bearer <JWT_TOKEN>
```

Tokens are obtained via:

- [`/api/auth/token`](auth/token.md) — local password flow
- [`/api/auth/ext/token`](auth/ext-token.md) — external-authenticator (mobile pairing) flow

Every token carries a `scope` claim that gates which endpoints it can invoke.

**Unauthenticated endpoints:** `/api/system/version`, `/api/system/status`, `/api/system/checkin`, `/api/auth/token`, `/api/auth/ext/token`, `/api/auth/ext/request`, `/api/auth/init`, and `/api/keymgmt/search` under the three-condition `allow_keysearch` bypass (see [search.md](keymgmt/search.md)).

Almost all endpoints require TLS; non-TLS requests return `418 I'm a teapot`.

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
| 409 | Incorrect internal state (fls_state, install-in-progress, not initialised) |
| 418 | TLS connection required |

## Endpoint Index

### System
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/system/version` | _none_ | [version.md](system/version.md) |
| GET | `/api/system/status` | _none_ | [status.md](system/status.md) |
| GET | `/api/system/checkin` | _none_ | [checkin.md](system/checkin.md) |
| POST | `/api/system/checkin` | _none_ | [checkin.md](system/checkin.md) |
| GET | `/api/system/config` | `system:config` (partial unauth) | [config.md](system/config.md) |
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

### Diagnostics (diagnostic-firmware builds only)
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/diag/*` (7 endpoints) | _varies_ | [diag.md](system/diag.md) |

### Authorization
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/auth/init` | _none_ | [init.md](auth/init.md) |
| POST | `/api/auth/init` | _none_ | [init.md](auth/init.md) |
| GET | `/api/auth/token` | _none_ | [token.md](auth/token.md) |
| POST | `/api/auth/token` | _none_ | [token.md](auth/token.md) |
| POST | `/api/auth/ext/init` | `auth:ext:pair` or `system:config` | [ext-init.md](auth/ext-init.md) |
| POST | `/api/auth/ext/validate` | `auth:ext:pair` | [ext-validate.md](auth/ext-validate.md) |
| POST | `/api/auth/ext/mac` | `auth:ext:pair` | [ext-mac.md](auth/ext-mac.md) |
| POST | `/api/auth/ext/request` | _none_ | [ext-request.md](auth/ext-request.md) |
| POST | `/api/auth/ext/token` | _none_ | [ext-token.md](auth/ext-token.md) |

### Key Management
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| POST | `/api/keymgmt/create` | `keymgmt:gen` | [create.md](keymgmt/create.md) |
| POST | `/api/keymgmt/derive` | `keymgmt:derive` / `keymgmt:gen` | [derive.md](keymgmt/derive.md) |
| POST | `/api/keymgmt/import` | `keymgmt:imp` | [import.md](keymgmt/import.md) |
| POST | `/api/keymgmt/update` | `keymgmt:upd` | [update.md](keymgmt/update.md) |
| DELETE | `/api/keymgmt/delete/:kid` | `keymgmt:del` | [delete.md](keymgmt/delete.md) |
| GET | `/api/keymgmt/get/:kid` | `keymgmt:get`/`keymgmt:gen`/`keymgmt:use:<kid>` | [get.md](keymgmt/get.md) |
| GET | `/api/keymgmt/list[/:offset[/:limit]]` | `keymgmt:list` | [list.md](keymgmt/list.md) |
| POST | `/api/keymgmt/search` | `keymgmt:search`/`keymgmt:list`/`auth:ext:pair` (or unauth bypass) | [search.md](keymgmt/search.md) |

### Storage (PPA only)
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/storage/unlock[/{ro\|rw}]` | `storage:disk<N>[:rw]` | [unlock.md](storage/unlock.md) |
| GET | `/api/storage/lock` | `storage:disk<N>` | [lock.md](storage/lock.md) |

### Audit Log
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| GET | `/api/logger/key` | `logger:get` | [key.md](logger/key.md) |
| GET | `/api/logger/list[/:offset]` | `logger:get` | [list.md](logger/list.md) |
| GET | `/api/logger/:id` | `logger:get` | [get.md](logger/get.md) |
| DELETE | `/api/logger/:id` | `logger:del` | [delete.md](logger/delete.md) (not dispatched in current firmware) |

### Cryptography
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| POST | `/api/crypto/hmac/hash` | `keymgmt:use:<kid>` | [hmac-hash.md](crypto/hmac-hash.md) |
| POST | `/api/crypto/hmac/verify` | `keymgmt:use:<kid>` | [hmac-verify.md](crypto/hmac-verify.md) |
| POST | `/api/crypto/exdsa/sign` | `keymgmt:use:<kid>` | [exdsa-sign.md](crypto/exdsa-sign.md) |
| POST | `/api/crypto/exdsa/verify` | `keymgmt:use:<kid>` | [exdsa-verify.md](crypto/exdsa-verify.md) |
| POST | `/api/crypto/ecdh` | `keymgmt:use:<kid>` | [ecdh.md](crypto/ecdh.md) |
| POST | `/api/crypto/cipher/encrypt` | `keymgmt:use:<kid>` | [cipher-encrypt.md](crypto/cipher-encrypt.md) |
| POST | `/api/crypto/cipher/decrypt` | `keymgmt:use:<kid>` | [cipher-decrypt.md](crypto/cipher-decrypt.md) |
| POST | `/api/crypto/cipher/wrap` | `keymgmt:use:<kid>` | [cipher-wrap.md](crypto/cipher-wrap.md) |
| POST | `/api/crypto/cipher/unwrap` | `keymgmt:use:<kid>` | [cipher-unwrap.md](crypto/cipher-unwrap.md) |

### Post-Quantum Cryptography
| Method | Endpoint | Scope | Doc |
|--------|----------|-------|-----|
| POST | `/api/crypto/pqc/mldsa/sign` | `keymgmt:use:<kid>` | [mldsa-sign.md](crypto/pqc/mldsa-sign.md) |
| POST | `/api/crypto/pqc/mldsa/verify` | `keymgmt:use:<kid>` | [mldsa-verify.md](crypto/pqc/mldsa-verify.md) |
| POST | `/api/crypto/pqc/mlkem/encaps` | `keymgmt:use:<kid>` | [mlkem-encaps.md](crypto/pqc/mlkem-encaps.md) |
| POST | `/api/crypto/pqc/mlkem/decaps` | `keymgmt:use:<kid>` | [mlkem-decaps.md](crypto/pqc/mlkem-decaps.md) |

---

## Cross-reference documents

- [FIRMWARE_NOTES.md](FIRMWARE_NOTES.md) — firmware bugs, undocumented behaviour, and other implementation gotchas.
- [discrepancies/DISCREPANCIES-OFFICIAL-DOCS.md](discrepancies/DISCREPANCIES-OFFICIAL-DOCS.md) — where the public Encedo docs disagree with reality.
- [discrepancies/DISCREPANCIES-ENCEDO-MANAGER.md](discrepancies/DISCREPANCIES-ENCEDO-MANAGER.md) — where the Encedo Manager client is internally inconsistent or diverges from firmware.
- [discrepancies/DISCREPANCIES-HEM-TEST.md](discrepancies/DISCREPANCIES-HEM-TEST.md) — behaviours the HEM test suite exercises that are missing from other sources.

## Concepts

- [concepts/device-options.md](concepts/device-options.md) — device-wide configuration flags (`allow_keysearch`, `trusted_ts`, etc.).
