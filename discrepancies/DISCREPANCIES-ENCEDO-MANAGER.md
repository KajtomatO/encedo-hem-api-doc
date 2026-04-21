# Discrepancies: Encedo Manager

Places where the [Encedo Manager](https://github.com/encedo/encedo-manager) (the reference web client) diverges from the firmware it talks to — either internally inconsistent, missing calls for endpoints it claims to support, or using a different call pattern than everybody else.

---

## 1. Logger DELETE scope mismatch

**Manager `_endpoints` registry ([build.js:9183](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9183), [scopes.js:720](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L720)):** Registers `DELETE api/logger/*` with scope `logger:get`.

**Manager consent system ([scopes.js:271-296](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L271)):** Defines a separate `logger:del` scope with "DELETE THE LOG" messaging.

**Firmware reality:** The `DELETE /api/logger/:id` handler exists but is **commented out of the dispatcher** — any request returns 404. So the scope-string mismatch is moot until the endpoint is re-enabled, but the inconsistency flags a bug worth fixing before that happens.

**Files:** [logger/delete.md](../logger/delete.md)

---

## 2. `keymgmt:usage:*` vs `keymgmt:use:*` naming drift

**Manager `_endpoints` map:** Uses `keymgmt:usage:*` in some places.

**Manager `scopes.js` consent UI:** Uses `keymgmt:use:{kid}#{descr}-{subtype}` (with regex matching).

**Firmware:** Accepts only **exact** `keymgmt:use:<kid_hex>` — no `keymgmt:usage:*` match. Tokens minted via [`/api/auth/ext/request`](../auth/ext-request.md) always use the `keymgmt:use:` form (plus the `#<base64-meta>` suffix).

**Impact:** Any code path in Manager that uses the `keymgmt:usage:*` string will never match a firmware scope check.

**Files:** [crypto/hmac-hash.md](../crypto/hmac-hash.md) (representative)

---

## 3. `/api/crypto/ecdh` not registered in Manager

Manager has no `_endpoints` entry, no `scopes.js` entry, and no call sites for `/api/crypto/ecdh`. The endpoint exists in firmware. Manager relies exclusively on [`/api/keymgmt/derive`](../keymgmt/derive.md) for key agreement.

**Files:** [crypto/ecdh.md](../crypto/ecdh.md)

---

## 4. `/api/system/shutdown` registered but never called

`scopes.js:89` defines a `system:shutdown` scope with consent messaging, but no Manager JavaScript invokes the endpoint. Likely reserved for third-party integrations.

**Files:** [system/shutdown.md](../system/shutdown.md)

---

## 5. `selftest` `repo_stats` sub-fields not consumed

Official docs for `/api/system/selftest` list `repo_stats` with sub-fields `total`, `deleted`, `invalid`, `fragmented`, `freeslots`. Manager reads `repo_stats` only from `/api/system/status`, not from `selftest`, so these sub-fields have no UI consumer.

---

## 6. PQC endpoints: keys creatable, operations not

Manager's key-creation UI supports `MLKEM512/768/1024` and `MLDSA44/65/87`, but the `mlkem/encaps`, `mlkem/decaps`, `mldsa/sign`, `mldsa/verify` endpoints are **not** wired into Manager code. Useful for third-party clients; not a Manager feature.

**Files:** [crypto/pqc/](../crypto/pqc/)

---

## 7. `/api/system/upgrade/upload_bootldr` partial wiring

**Manager `_endpoints`:** Registers the upload endpoint ([build.js:9142](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9142)).

**Manager UI:** Has an `update_bootloader_page` HTML block, but no `check_bootldr` / `install_bootldr` counterparts and no JS calling the upload endpoint from the UI.

Unlike firmware upgrades (`upload_fw` → `check_fw` → `install_fw`), the bootloader flow stops at upload. Either the verification/install lives elsewhere (e.g., boot-time) or the feature is incomplete.

**Files:** [system/upgrade-bootloader.md](../system/upgrade-bootloader.md)

---

## 8. Storage unlock `/ro` and `/rw` suffixes are Manager-only

Manager invariably appends `/ro` or `/rw` when calling `/api/storage/unlock` ([build.js:7729-7809](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7729)). The test suite and the original docs use the plain path. Firmware accepts both — the suffix, when present, overrides the scope's mode bit. So this is a convention, not a requirement.

**Files:** [storage/unlock.md](../storage/unlock.md)

---

## 9. Duplicate entries in `_endpoints` / `scopes.js`

`/api/crypto/exdsa/verify` appears twice in `build.js` ([:9268](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9268) and [:9273](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9273)) and in `scopes.js` ([:805](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L805) and [:810](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L810)). JS object-key collision means the second entry overwrites the first — harmless but noise.

**Files:** [crypto/exdsa-verify.md](../crypto/exdsa-verify.md)
