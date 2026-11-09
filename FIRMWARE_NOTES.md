# Firmware Notes

Observations about the device firmware that are not part of the stated API contract but affect how implementers should interact with it — suspected bugs, disabled endpoints, subtle behaviours, undocumented side-effects. Each note links to the relevant endpoint page.

This file is a living catalogue: add new entries when you find behaviour that a careful integrator would want to know about but that doesn't belong in the per-endpoint reference.

---

## 1. `DELETE /api/logger/:id` handler exists but is not dispatched

The firmware contains a `api_delete_logger_id` implementation, but the dispatcher entry is commented out with a `/* not available in CC mode */` note. Any DELETE request to `/api/logger/<id>` on current firmware returns 404. The [logger/delete.md](logger/delete.md) page documents the handler's eventual shape; treat the endpoint as unavailable until a firmware build re-enables it.

Cross-reference: [logger/delete.md](logger/delete.md), [discrepancies/DISCREPANCIES-ENCEDO-MANAGER.md](discrepancies/DISCREPANCIES-ENCEDO-MANAGER.md) (Manager has the endpoint in its registry with conflicting scope strings).

---

## 2. `/api/keymgmt/get` deliberately omits `label`

The repo stores a `label` on every key and returns it in list / search responses, but the GET-by-kid handler **does not** surface it — the `cJSON_AddStringToObject(json, "label", label)` call is commented out. `type`, `pubkey`, `der`, `descr`, and `updated` are all returned.

If you need the label for a specific kid, enumerate via [`/api/keymgmt/list`](keymgmt/list.md) or search on `descr` via [`/api/keymgmt/search`](keymgmt/search.md).

Cross-reference: [keymgmt/get.md](keymgmt/get.md).

---

## 3. ExtAuth `keymgmt:use:<kid>` scope is rewritten during request, not token

[`/api/auth/ext/request`](auth/ext-request.md) rewrites the requested scope before packaging it for the paired authenticator: if the client asked for `keymgmt:use:<kid>`, the firmware appends `#<base64({"t":"<type_hex>","l":"<label>"})>` so the authenticator UI can show a meaningful consent prompt. The final token minted by [`/api/auth/ext/token`](auth/ext-token.md) still contains the full `keymgmt:use:<kid>#<meta>` string, but the scope-check handlers (`/api/crypto/*`, `/api/keymgmt/get`) do **exact** `strcmp` against `keymgmt:use:<kid>` — they strip the `#<meta>` suffix before matching.

Callers who parse the token's `scope` claim themselves should split on `#` and use only the prefix for authorization decisions.

Separately, `keymgmt:use:<kid>` tokens are minted with a **15-minute** lifetime instead of the default 60 minutes. Plan renewals accordingly.

Cross-reference: [auth/ext-request.md](auth/ext-request.md), [auth/ext-token.md](auth/ext-token.md).

---

## 4. HMAC hash and verify share one audit event type

Both `api_post_crypto_hmac_hash` and `api_post_crypto_hmac_verify` emit `LOG_TYPE_CRYPTO_HMAC_HASH` (no separate `HMAC_VERIFY` type in the enum). Verifiers of audit logs cannot distinguish the two operations from the event type alone — only the `LOG_RESULT_OK/FAILED` sub-field and the surrounding timing are available.

Cross-reference: [crypto/hmac-hash.md](crypto/hmac-hash.md), [crypto/hmac-verify.md](crypto/hmac-verify.md).

---

## 5. `/api/crypto/cipher/encrypt` always generates its own IV

Despite some external docs suggesting an `iv` request field, the encrypt handler ignores caller-supplied IVs and always generates a fresh random 16-byte IV via `my_rng_gen_block`. The generated IV is returned in the response for CBC / GCM so the caller can hand it to `decrypt` later. ECB gets no IV in the response (as expected).

Cross-reference: [crypto/cipher-encrypt.md](crypto/cipher-encrypt.md).

---

## 6. `alg` is effectively required on `/api/crypto/cipher/encrypt` and `/decrypt`

The parser's default is `AES256-GCM`, but the code path only accepts an `alg` string of length exactly 10 — any shorter or longer value, including missing, falls into the `goto ONFAIL` branch (400). So the "default" is cosmetic; callers must send one of the nine exact `AES<128|192|256>-<ECB|CBC|GCM>` strings.

Cross-reference: [crypto/cipher-encrypt.md](crypto/cipher-encrypt.md), [crypto/cipher-decrypt.md](crypto/cipher-decrypt.md).

---

## 7. `/api/keymgmt/search` unauthenticated bypass requires three conditions

The unauthenticated `^EXTAID` lookup that Manager uses to discover paired authenticators works only when all three hold simultaneously:

1. Query is prefix-anchored (`^...`).
2. Decoded prefix is **≥ 6 bytes** (base64 `^RVhUQUlE` meets this).
3. Device-level [`allow_keysearch`](concepts/device-options.md#allow_keysearch--enable-unauthenticated-key-search) is enabled.

Any other shape requires a Bearer token. Devices shipped with `allow_keysearch` disabled will reject the bypass even for well-formed `^EXTAID` queries, which breaks the pairing-discovery path.

Cross-reference: [keymgmt/search.md](keymgmt/search.md).

---

## 8. ML-KEM implicit rejection masquerades as success

`CRYPTO_MLKEM_Decaps` never returns a "ciphertext invalid" error — ML-KEM's implicit rejection design produces a pseudo-random shared secret for malformed / forged ciphertexts and proceeds. A `200 OK` from [`/api/crypto/pqc/mlkem/decaps`](crypto/pqc/mlkem-decaps.md) does **not** mean the ciphertext was authentic. Pair with an HMAC or signature on a derived key to detect tampering.

Cross-reference: [crypto/pqc/mlkem-decaps.md](crypto/pqc/mlkem-decaps.md).

---

## 9. ECDH key derivation info labels are fixed per type

[`/api/keymgmt/derive`](keymgmt/derive.md) uses HKDF-SHA256 with a hard-coded `info` label of the form `encedo-<type_lowercase>` (e.g. `encedo-aes256`, `encedo-secp256r1`). Two parties that agree on `type` and exchange public keys end up with the same derived key without any further handshake — useful for pair-wise symmetric channels. Conversely, deriving two different `type`s against the same peer gives independent keys because the `info` differs.

PQC types (`MLKEM*`, `MLDSA*`) are **not** supported by derive.

Cross-reference: [keymgmt/derive.md](keymgmt/derive.md).

---

## 10. KDF used at init sets the lock-in for life

The firmware is KDF-agnostic at auth-check time. It never sees the password or the KDF; it only compares the public key in the JWT's `iss` claim with the UserKey/MasterKey stored at [`/api/auth/init`](auth/init.md). Those keys cannot be changed without re-initialising. Whichever KDF a client picks must therefore be used consistently for every subsequent [`/api/auth/token`](auth/token.md) call. Encedo Manager and the test suite both use PBKDF2-HMAC-SHA256 (600 000 iterations, salt = `eid` string); a custom client may choose another. Mixing KDFs silently fails: a different KDF yields a different keypair, so `iss` matches no stored key.

Cross-reference: [auth/init.md](auth/init.md), [auth/token.md](auth/token.md), [discrepancies/DISCREPANCIES-HEM-TEST.md](discrepancies/DISCREPANCIES-HEM-TEST.md).

---

## 11. `POST /api/keymgmt/list` in Manager's registry is aspirational

Manager's endpoint map lists `POST api/keymgmt/list` as an "extended version", but the firmware `keymgmt` router implements only `GET /api/keymgmt/list[/:offset[/:limit]]`. POSTing to that path returns an error. Treat the Manager entry as unimplemented.

Cross-reference: [keymgmt/list.md](keymgmt/list.md).

---

## 12. External authenticator cap

The ExtAuth subsystem holds at most `EXT_AUTH_MAX_CLIENTS` (currently 8) paired authenticators simultaneously. [`/api/auth/ext/init`](auth/ext-init.md) against a full slot table replaces an existing slot rather than growing unbounded. Callers that need more than 8 paired devices cannot get them from this API.

Cross-reference: [auth/ext-init.md](auth/ext-init.md).
