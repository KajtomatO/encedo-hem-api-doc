# Official Documentation vs Encedo Manager: Discrepancies

This document catalogs every instance where the [official Encedo HEM API documentation](https://docs.encedo.com/hem-api/reference/api-reference) differs from what the [Encedo Manager](https://github.com/encedo/encedo-manager) actually implements, or where information is missing from the official docs.

---

## Critical Discrepancies

### 1. Storage unlock URL paths undocumented

**Official docs:** `/api/storage/unlock` -- disk and mode derived from JWT scope.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Uses `/api/storage/unlock/ro` and `/api/storage/unlock/rw` path suffixes ([build.js:7729-7809](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7729), [core2.js:5437-5517](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5437)). The older [encedo.js:1691](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1691) uses the plain `/api/storage/unlock`, but the newer `build.js` consistently uses the suffixed versions.

**Impact:** Any implementation following the official docs would use the wrong URL path.

**Files:** [storage/unlock.md](storage/unlock.md)

---

### 2. `/api/system/config` GET works without auth during pairing

**Official docs:** `Authorization: Bearer JWT_TOKEN` is marked as required for GET.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** The `pair()`, `paired()`, and `unpair()` methods ([build.js:383](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L383), [build.js:419](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L419), [build.js:455](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L455)) call `GET /api/system/config` without authentication to retrieve the `eid` field. The device returns at least a subset of config data (specifically `eid`) without a token.

**Impact:** The docs imply auth is always required, but the pairing flow depends on unauthenticated access to `eid`.

**Files:** [system/config.md](system/config.md)

---

### 3. Logger DELETE scope mismatch in endpoint registry

**Official docs:** Not explicitly documented.

**[Encedo Manager](https://github.com/encedo/encedo-manager) `_endpoints` registry:** [build.js:9183](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9183) and [scopes.js:720](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L720) register `DELETE api/logger/*` with scope `logger:get`.

**Encedo Manager scope consent system:** [scopes.js:271-296](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L271) defines a separate `logger:del` scope with "DELETE THE LOG" / "Log file deleted" messaging.

**Impact:** The endpoint registry says `logger:get` but the consent UI uses `logger:del`. This is likely a bug in the endpoint registry.

**Files:** [logger/delete.md](logger/delete.md)

---

## Missing from Official Docs

### 4. `/api/system/upgrade/upload_bootldr` -- Incomplete upgrade triad

**Official docs:** Not documented.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Registered in `_endpoints` ([build.js:9142](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9142)) with scope `system:upgrade`. However, unlike firmware (upload/check/install) and UI (upload/check/install), there are no corresponding `check_bootldr` or `install_bootldr` endpoints. The HTML has an `update_bootloader_page` but no JS logic.

**Impact:** Bootloader updates appear partially implemented or handled differently.

**Files:** [system/upgrade-bootloader.md](system/upgrade-bootloader.md)

---

### 5. `/api/crypto/cipher/*` endpoints -- Official docs return 404 (RESOLVED)

**Official docs:** The documentation pages for encryption/decryption and wrap/unwrap return **404 errors**.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Four endpoints are registered but no request/response schemas in Manager code.

**Resolution:** Full schemas recovered from [HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_11.php](https://github.com/encedo/hem-api-tester/blob/main/test_11.php)). See discrepancy #17 for details.

**Files:** [crypto/cipher-encrypt.md](crypto/cipher-encrypt.md), [crypto/cipher-decrypt.md](crypto/cipher-decrypt.md), [crypto/cipher-wrap.md](crypto/cipher-wrap.md), [crypto/cipher-unwrap.md](crypto/cipher-unwrap.md)

---

### 6. `/api/crypto/ecdh` -- Not in Encedo Manager

**Official docs:** Documented with full request/response schemas.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Not registered in `_endpoints`, not in `scopes.js`, not called anywhere in the Manager JavaScript. Key derivation is handled via `/api/keymgmt/derive` instead.

**Impact:** The endpoint likely exists on the device but the Manager UI uses a different approach.

**Files:** [crypto/ecdh.md](crypto/ecdh.md)

---

### 7. POST variant of `/api/keymgmt/list` undocumented

**Official docs:** Only the `GET /api/keymgmt/list/:offset/:limit` variant is documented.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** The `_endpoints` map ([build.js:9195](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9195)) registers `POST /api/keymgmt/list` described as "List stored keys (extended version)" with scope `keymgmt:list`. The POST variant likely accepts filter criteria in the request body, but neither the official docs nor the Manager code reveal the body schema.

**Impact:** The extended list functionality is unknown.

**Files:** [keymgmt/list.md](keymgmt/list.md)

---

### 8. External authenticator registration/authentication docs -- Broken URLs

**Official docs:** The URLs `authorization/registration-external` and `authorization/authentication-external` both return **404**. The correct URLs use a different path structure: `authorization/external-authenticator/registration` and `authorization/external-authenticator/authentication`.

**Impact:** Navigation from the API reference index page leads to dead links for external authenticator documentation.

---

## Minor Discrepancies

### 9. JWT structure undocumented

**Official docs:** The `auth` field in `POST /api/auth/token` and `init` field in `POST /api/auth/init` are described as opaque strings.

**[Encedo Manager](https://github.com/encedo/encedo-manager) reveals:** These are JWTs signed with HMAC-SHA256 using an X25519 ECDH-derived shared secret. The JWT header is `{ "ecdh": "x25519" }` and the payload contains specific claims (`jti`, `aud`, `exp`, `iat`, `iss`, `scope`, `cfg`). The key derivation uses Argon2 (time=10, mem=8192, hashLen=32) with the device's `eid` as salt.

**Impact:** The official docs provide no guidance on how to construct the authentication JWT, making it impossible to implement authentication without the Encedo Manager source code.

---

### 10. `/api/system/shutdown` -- No direct call in Manager

**Official docs:** Documented as a standard endpoint.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** The scope `system:shutdown` is defined in [scopes.js:89](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L89) but no JavaScript code calls `/api/system/shutdown` directly. The consent dialog exists, suggesting it's used by third-party apps or an unused feature.

---

### 11. Scope naming inconsistency

**Official docs / endpoint registry:** Uses `keymgmt:usage:*` for crypto operation scopes.

**[Encedo Manager](https://github.com/encedo/encedo-manager) consent system (`scopes.js`):** Uses `keymgmt:use:{kid}#{descr}-{subtype}` pattern with regex matching for key-specific scopes.

The `_endpoints` map uses `keymgmt:usage:*` while `scopes.js` uses `keymgmt:use:*` -- these are different strings. The actual scope sent to the device follows the `scopes.js` pattern.

---

### 12. `selftest` response -- `repo_stats` sub-fields

**Official docs:** Lists `repo_stats` with sub-fields (total, deleted, invalid, fragmented, freeslots).

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Does not reference these `repo_stats` sub-fields from the selftest response. The `repo_stats` object appears in `/api/system/status` response handling instead.

---

## Documentation Quality Issues

### 13. Official docs lack request/response examples for many endpoints

Endpoints that return empty response bodies in the official docs but likely return confirmation data:
- `POST /api/system/config` -- only partially documents the response
- `DELETE /api/keymgmt/delete/:kid` -- no response body documented
- `POST /api/keymgmt/update` -- no response body documented
- `GET /api/storage/unlock` -- no response body documented
- `GET /api/storage/lock` -- no response body documented

---

### 14. ML-DSA and ML-KEM crypto operations -- Documented but not in Encedo Manager

The official docs document four post-quantum crypto endpoints under `/api/crypto/pqc/`:
- `POST /api/crypto/pqc/mldsa/sign` -- ML-DSA signing (FIPS 204)
- `POST /api/crypto/pqc/mldsa/verify` -- ML-DSA verification
- `POST /api/crypto/pqc/mlkem/encaps` -- ML-KEM encapsulation (FIPS 203)
- `POST /api/crypto/pqc/mlkem/decaps` -- ML-KEM decapsulation

These are **not present** in the [Encedo Manager](https://github.com/encedo/encedo-manager) `_endpoints` map or `scopes.js`, confirming they are newer firmware features not yet integrated into the Manager UI. The key types (MLKEM512/768/1024, MLDSA44/65/87) are already listed in `/api/keymgmt/create` documentation, so keys can be generated -- but the PQC crypto operations have no UI.

Note: These endpoints use a different path prefix (`/api/crypto/pqc/`) compared to classical crypto (`/api/crypto/`). The official docs are **fully sufficient** to implement these -- request/response schemas, algorithms, and scopes are all documented.

See: [crypto/pqc/](crypto/pqc/)

---

## Test Suite vs Manager / Official Docs

_The following discrepancies were discovered by analyzing the [HEM API test suite](https://github.com/encedo/hem-api-tester) against our API documentation and the [Encedo Manager](https://github.com/encedo/encedo-manager)._

### 15. KDF mismatch: Argon2 (Manager) vs PBKDF2 (test suite)

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Uses **Argon2** (time=10, mem=8192, hashLen=32) with device `eid` as salt to derive the 32-byte secret from `password`.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([libs/lib.php](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php)):** Uses **PBKDF2-SHA256** (600,000 iterations, 32 bytes) with device `eid` as salt.

Both derive a 32-byte secret used identically: NaCl X25519 keypair → ECDH with device `spk` → HMAC-SHA256 JWT signing. The device accepts tokens from both KDFs, meaning authentication is KDF-agnostic as long as the same password-derived key material was registered during init.

**Impact:** An implementation must use whichever KDF was used during initialization. Using the wrong KDF produces a different key and authentication will fail.

**Files:** [auth/token.md](auth/token.md), [auth/init.md](auth/init.md)

---

### 16. JWT header discrepancy: Manager vs test suite

**Manager:** Sets JWT header as `{ "ecdh": "x25519" }`.

**Test suite:** Sets JWT header as `{ "alg": "HS256", "typ": "JWT", "ecdh": "x25519" }`.

Both are accepted by the device, indicating the device only checks the `ecdh` field and ignores standard JWT header fields.

**Files:** [auth/token.md](auth/token.md), [auth/init.md](auth/init.md)

---

### 17. Cipher endpoint schemas -- Official docs 404, recovered from test suite

**Official docs:** All four cipher endpoint documentation pages return **404 errors**.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_11.php](https://github.com/encedo/hem-api-tester/blob/main/test_11.php)):** Provides full working schemas:
- **encrypt:** Request `{kid, msg, alg, iv, aad}` → Response `{ciphertext, iv, tag}`
- **decrypt:** Request `{kid, msg, alg, iv, tag}` → Response `{plaintext}`
- **wrap:** Request `{kid, alg, msg}` → Response `{wrapped}`
- **unwrap:** Request `{kid, alg, msg}` → Response `{unwrapped}`

Supported algorithms: AES128/192/256-ECB, AES128/192/256-CBC, AES128/192/256-GCM.

**Files:** [crypto/cipher-encrypt.md](crypto/cipher-encrypt.md), [crypto/cipher-decrypt.md](crypto/cipher-decrypt.md), [crypto/cipher-wrap.md](crypto/cipher-wrap.md), [crypto/cipher-unwrap.md](crypto/cipher-unwrap.md)

---

### 18. Key type naming: no dashes for AES, no prefix for HMAC

**Our original docs (from official docs):** Listed AES key types as `AES-128`, `AES-192`, `AES-256` and HMAC types as `HMAC-SHA2-256`, etc.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_10.php](https://github.com/encedo/hem-api-tester/blob/main/test_10.php)):** Uses `AES128`, `AES192`, `AES256` (no dash) and `SHA2-256`, `SHA3-512` (no `HMAC-` prefix) as the `type` values passed to `/api/keymgmt/create`.

**Impact:** Using the wrong type string (e.g., `AES-128` instead of `AES128`) would cause a 400 error.

**Files:** [keymgmt/create.md](keymgmt/create.md)

---

### 19. Storage unlock: test suite uses plain path (no /ro /rw suffix)

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Uses `/api/storage/unlock/ro` and `/api/storage/unlock/rw`.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_4.php](https://github.com/encedo/hem-api-tester/blob/main/test_4.php)):** Uses plain `/api/storage/unlock` without suffix, aligning with the official docs and endpoint registry.

**Impact:** The `/ro` and `/rw` suffixes appear to be a Manager UI convention. The device accepts the plain path, with access mode determined by the JWT scope.

**Files:** [storage/unlock.md](storage/unlock.md)

---

### 20. `/api/system/upgrade/usbmode` -- Undocumented endpoint

**Official docs:** Not documented.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Not present in `_endpoints` or any JavaScript.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_7.php:90](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L90), [upgrade2diag.php:76](https://github.com/encedo/hem-api-tester/blob/main/upgrade2diag.php#L76)):** Uses `GET /api/system/upgrade/usbmode` with scope `system:upgrade` to put the device into USB serial mode for firmware upload via serial port (Intel HEX format).

**Files:** [system/upgrade-usbmode.md](system/upgrade-usbmode.md)

---

### 21. `/api/diag/*` -- 7 diagnostic endpoints (diagnostic firmware only)

**Official docs:** Not documented.

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Not present.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([test_8.php](https://github.com/encedo/hem-api-tester/blob/main/test_8.php)):** Uses 7 diagnostic endpoints (`break_tls`, `break_trng`, `test_trng`, `memdump`, `corrupt_repo`, `wipe_config`, `break_temp`) available only on `-DIAG` firmware builds. These are fault injection tools for hardware security testing.

**Files:** [system/diag.md](system/diag.md)

---

### 22. Audit log integrity verification -- Ed25519 + HMAC-SHA256 chain

**Official docs:** Do not describe the log integrity verification mechanism.

**[HEM API test suite](https://github.com/encedo/hem-api-tester) ([libs/lib.php](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php)):** Implements full log chain verification: each log entry is signed with Ed25519, and entries are chained via HMAC-SHA256 (each entry's HMAC includes the previous entry's HMAC). The logger public key from `GET /api/logger/key` is used to verify Ed25519 signatures.

**Impact:** Without this knowledge, an implementation cannot verify audit log integrity.

**Files:** [logger/key.md](logger/key.md), [logger/get.md](logger/get.md)

