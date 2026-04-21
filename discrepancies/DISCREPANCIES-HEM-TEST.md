# Discrepancies: HEM API Test Suite

Behaviours, endpoints, and mechanics that the [HEM API test suite](https://github.com/encedo/hem-api-tester) uses or exposes, but which differ from the Encedo Manager client or are absent from the public docs. The test suite is a good oracle for firmware-level truth because it covers more of the surface than Manager does.

---

## 1. KDF choice is client-side — Argon2 in Manager, PBKDF2 in test suite

**[Encedo Manager](https://github.com/encedo/encedo-manager):** Derives the 32-byte authentication secret via **Argon2** (`time=10`, `mem=8192`, `hashLen=32`) with the device `eid` as salt.

**[Test suite](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php):** Derives the same slot via **PBKDF2-SHA256** (600 000 iterations, 32 bytes) with the same `eid` salt.

Both feed the derived secret into an identical downstream path — NaCl X25519 keypair → ECDH with device `spk` → HMAC-SHA256 JWT signing. The firmware does not care which KDF was used; it only validates that the resulting shared secret matches the one registered at init time. Practical consequence: an implementation must stick with whichever KDF was used during `/api/auth/init`. Mixing them means authentication fails.

**Files:** [auth/token.md](../auth/token.md), [auth/init.md](../auth/init.md)

---

## 2. JWT header fields are not enforced

**Manager:** Sets header `{"ecdh": "x25519"}`.

**Test suite:** Sets header `{"alg": "HS256", "typ": "JWT", "ecdh": "x25519"}`.

Both are accepted — firmware reads only the `ecdh` field and ignores standard JWT header fields. The `alg` and `typ` keys are documentation convenience, not validation gates.

**Files:** [auth/token.md](../auth/token.md), [auth/init.md](../auth/init.md)

---

## 3. Cipher endpoint schemas recovered from test suite

With the docs pages returning 404, [`test_11.php`](https://github.com/encedo/hem-api-tester/blob/main/test_11.php) is the public source for the request/response shapes:

- **encrypt:** Request `{kid, msg, alg, aad?, ext_kid?, pubkey?, ctx?}` → Response `{ciphertext, iv, tag}` (IV is server-generated; see [`cipher-encrypt.md`](../crypto/cipher-encrypt.md))
- **decrypt:** Request `{kid, msg, alg, iv, tag?, aad?, ext_kid?, pubkey?, ctx?}` → Response `{plaintext}`
- **wrap:** Request `{kid, msg, alg?, ext_kid?, pubkey?, ctx?, iv?}` → Response `{wrapped}`
- **unwrap:** Request `{kid, msg, alg?, ext_kid?, pubkey?, ctx?, iv?}` → Response `{unwrapped}`

Supported algorithms: `AES128/192/256-{ECB,CBC,GCM}` for encrypt/decrypt; `AES128/192/256` for wrap/unwrap. The ECDH-derived-KEK variants (`ext_kid` / `pubkey` / `ctx`) are present in firmware but not exercised by every test.

**Files:** [crypto/cipher-encrypt.md](../crypto/cipher-encrypt.md), [crypto/cipher-decrypt.md](../crypto/cipher-decrypt.md), [crypto/cipher-wrap.md](../crypto/cipher-wrap.md), [crypto/cipher-unwrap.md](../crypto/cipher-unwrap.md)

---

## 4. Key type strings use no dashes for AES, no `HMAC-` prefix

**Old/official docs:** Suggested `AES-128`, `HMAC-SHA2-256`, etc.

**[test_10.php](https://github.com/encedo/hem-api-tester/blob/main/test_10.php):** Uses `AES128`, `AES192`, `AES256`, and `SHA2-256`, `SHA3-512` (no `HMAC-` prefix) as the `type` field passed to [`/api/keymgmt/create`](../keymgmt/create.md).

Using the wrong form returns 400.

**Files:** [keymgmt/create.md](../keymgmt/create.md)

---

## 5. `/api/storage/unlock` — test suite uses the plain path

**Manager:** Always calls `/api/storage/unlock/ro` or `/api/storage/unlock/rw`.

**[test_4.php](https://github.com/encedo/hem-api-tester/blob/main/test_4.php):** Calls plain `/api/storage/unlock` and lets the scope decide the mode.

Firmware supports both (suffix overrides scope when present). Good orthogonality note for implementers.

**Files:** [storage/unlock.md](../storage/unlock.md)

---

## 6. `/api/system/upgrade/usbmode` — only visible in test suite

**Official docs:** Not documented.

**Manager:** Not registered or called.

**[test_7.php:90](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L90), [upgrade2diag.php:76](https://github.com/encedo/hem-api-tester/blob/main/upgrade2diag.php#L76):** `GET /api/system/upgrade/usbmode` with scope `system:upgrade` — puts the device into USB serial mode for firmware upload via Intel HEX over UART. Lab / recovery tool.

**Files:** [system/upgrade-usbmode.md](../system/upgrade-usbmode.md)

---

## 7. `/api/diag/*` — diagnostic firmware only

**Official docs:** Not documented.

**Manager:** Not present.

**[test_8.php](https://github.com/encedo/hem-api-tester/blob/main/test_8.php):** Uses seven diagnostic / fault-injection endpoints (`break_tls`, `break_trng`, `test_trng`, `memdump`, `corrupt_repo`, `wipe_config`, `break_temp`). Available only in `-DIAG` firmware builds used for security testing; standard firmware does not compile them in.

**Files:** [system/diag.md](../system/diag.md)

---

## 8. Audit log chain verification (Ed25519 + HMAC-SHA256)

**Official docs:** Silent on integrity verification.

**[libs/lib.php](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php):** Implements full chain verification — each log entry has an Ed25519 signature produced by the firmware's `EIDkeySign` (served via [`/api/logger/key`](../logger/key.md)), and entries are linked via HMAC-SHA256 where each HMAC input includes the previous entry's HMAC. Breaking the chain (missing entries, tampering) is detectable by recomputing.

Third-party verifiers need the logger key exactly once (pinned by `eid` + Ed25519 pubkey) and can then audit the whole log offline.

**Files:** [logger/key.md](../logger/key.md), [logger/get.md](../logger/get.md)
