# Discrepancies: Official Documentation

Gaps, errors, and missing sections in the [official Encedo HEM API reference](https://docs.encedo.com/hem-api/reference/api-reference). For cases where something was missing from the docs but could be recovered from other sources (firmware behaviour, Manager source, test suite), the resolution is noted per item.

---

## 1. Storage unlock URL paths undocumented

**Official docs:** `/api/storage/unlock` — disk and mode derived from JWT scope only.

**Actual firmware:** Accepts `/api/storage/unlock`, `/api/storage/unlock/ro`, and `/api/storage/unlock/rw`. Path suffix, when present, overrides the mode bit extracted from the scope (scope still selects the disk index).

**Impact:** Docs-following implementations cannot express "force read-only on a scope minted RW". Manager uses the suffixed form; the test suite uses the plain form — both work.

**Files:** [storage/unlock.md](../storage/unlock.md)

---

## 2. `GET /api/system/config` works without auth during pairing

**Official docs:** Bearer token marked as required.

**Actual firmware:** Responds with the `eid` portion (and a subset of identifying config) on unauthenticated GETs so that pairing / discovery clients can learn the device identifier before any JWT exists. Full config fields still require auth.

**Impact:** Without this carve-out, the Manager's `pair()` / `paired()` / `unpair()` flows would be impossible — they need `eid` to construct the ECDH-based JWT in the first place.

**Files:** [system/config.md](../system/config.md)

---

## 3. External authenticator reference URLs return 404

`authorization/registration-external` and `authorization/authentication-external` are dead links on the public docs site. The live sections live at `authorization/external-authenticator/registration` and `authorization/external-authenticator/authentication`.

**Files:** [auth/ext-init.md](../auth/ext-init.md), [auth/ext-validate.md](../auth/ext-validate.md), [auth/ext-request.md](../auth/ext-request.md), [auth/ext-token.md](../auth/ext-token.md)

---

## 4. JWT construction not documented

The `auth` / `init` payloads of `/api/auth/token` and `/api/auth/init` are described as opaque strings. In reality they are full JWTs:

- HMAC-SHA256 signed over an ECDH-derived shared secret
- Header includes `{"ecdh": "x25519"}`
- Payload carries `jti`, `aud`, `exp`, `iat`, `iss`, `scope`, `cfg` claims

The 32-byte secret can be derived via any KDF the client picks (both Manager and the test suite use PBKDF2-HMAC-SHA256, 600 000 iterations, salt = `eid`) — the firmware only checks that the resulting key matches the one registered at init.

**Files:** [auth/token.md](../auth/token.md), [auth/init.md](../auth/init.md)

---

## 5. Incomplete bootloader upgrade triad

**Official docs:** Not documented.

**Actual firmware:** A bootloader-upload endpoint exists (`/api/system/upgrade/upload_bootldr`) but has no corresponding `check_bootldr` / `install_bootldr` counterparts like the firmware triad. Manager has an HTML page but no JS wiring.

**Files:** [system/upgrade-bootloader.md](../system/upgrade-bootloader.md)

---

## 6. `/api/crypto/cipher/*` pages return 404

All four cipher endpoint pages on the public docs site are broken:

- `/api/crypto/cipher/encrypt`
- `/api/crypto/cipher/decrypt`
- `/api/crypto/cipher/wrap`
- `/api/crypto/cipher/unwrap`

**Resolution:** Schemas reconstructed from firmware behaviour and the [HEM API test suite](https://github.com/encedo/hem-api-tester/blob/main/test_11.php). See [`crypto/cipher-encrypt.md`](../crypto/cipher-encrypt.md) et al. for the full request/response shapes — including the ECDH-derived-KEK variants (`ext_kid` / `pubkey` / `ctx`) that the official docs do not describe.

---

## 7. `/api/crypto/ecdh` documented but semantics fuzzy

The official docs name the endpoint but do not clearly distinguish it from [`/api/keymgmt/derive`](../keymgmt/derive.md). Firmware truth:

- `crypto/ecdh` returns the **raw** ECDH output (optionally hashed with `alg`) to the caller and stores nothing.
- `keymgmt/derive` feeds the ECDH output through HKDF-SHA256 with an algorithm-specific `info` label and stores a fresh derived key in the repo.

**Files:** [crypto/ecdh.md](../crypto/ecdh.md), [keymgmt/derive.md](../keymgmt/derive.md)

---

## 8. `POST /api/keymgmt/list` variant undocumented

**Official docs:** Only `GET /api/keymgmt/list/:offset/:limit`.

**Endpoint registry:** `POST api/keymgmt/list` is listed as "extended version" with scope `keymgmt:list`.

**Actual firmware:** The current firmware `keymgmt` router does not implement a POST handler for `list` — only GET. The POST entry in Manager appears to be aspirational or legacy. Treat it as unavailable until firmware support lands.

**Files:** [keymgmt/list.md](../keymgmt/list.md)

---

## 9. PQC endpoints documented but not wired into Manager

`/api/crypto/pqc/mldsa/{sign,verify}` and `/api/crypto/pqc/mlkem/{encaps,decaps}` are fully documented in the public docs and implemented in firmware, but Encedo Manager has no UI for them. Key generation (which *is* in Manager) produces usable ML-KEM / ML-DSA keys, but you need a third-party client to exercise the crypto operations.

**Files:** [crypto/pqc/](../crypto/pqc/)

---

## 10. Many endpoints lack response-body documentation

Endpoints where the docs leave the success response under-specified (firmware behaviour is filled in under each file):

- `POST /api/system/config` — partial response documentation
- `DELETE /api/keymgmt/delete/:kid` — no body documented (firmware: `200 OK`, empty)
- `POST /api/keymgmt/update` — no body documented (firmware: `200 OK`, empty)
- `GET /api/storage/unlock` — no body documented
- `GET /api/storage/lock` — no body documented

---

## 11. `/api/system/shutdown` exists in docs, no UI

Documented endpoint with matching scope (`system:shutdown`) defined in Manager's `scopes.js:89` but no JavaScript call site — suggesting it's reserved for third-party consumers, not a Manager feature.

**Files:** [system/shutdown.md](../system/shutdown.md)
