# POST /api/keymgmt/import

Import a **public key** (external peer) into the on-device key repository so subsequent calls can reference it by `kid` — e.g. as the counter-party in [`/api/crypto/ecdh`](../crypto/ecdh.md), as a verifier in `exdsa-verify`, or as the recipient in `cipher-wrap`.

Only *public* key material is accepted; there is no way to import a private key.

## Request

```
POST /api/keymgmt/import
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** prefix-match `keymgmt:imp`.
- **Subject:** `sub != "M"`.

**Preconditions:** same as [`/api/keymgmt/create`](create.md) — `fls_state==0`, no software install, valid JWT, TLS, initialised.

### Body

```json
{
  "type":   "<algorithm>",
  "label":  "<human_label>",
  "pubkey": "<base64_public_key>",
  "mode":   "ECDH" | "ECDH,ExDSA" | "ExDSA",
  "descr":  "<base64_opaque_blob>"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `type` | String | Yes | Algorithm name — see below. |
| `label` | String | Yes | `isvalid_label` ≤32 bytes. |
| `pubkey` | String | Yes | Base64 of the raw public key. Decoded length ≤70 bytes (room for NIST P-521 + overhead). |
| `mode` | String | No | NIST-P ECC only — sets role bits. |
| `descr` | String | No | Base64 blob, ≤`STOREDKEY_DESCR_MAX_LENGTH` bytes decoded. |

### Supported `type` values (asymmetric only)

`SECP256R1`, `SECP384R1`, `SECP521R1`, `SECP256K1`, `CURVE25519`, `CURVE448`, `ED25519`, `ED448`, `MLKEM512`, `MLKEM768`, `MLKEM1024`, `MLDSA44`, `MLDSA65`, `MLDSA87`.

Symmetric types (HMAC/AES) cannot be imported — that would require smuggling key material, which the repo rejects by design.

## Response

**Status:** `200 OK`

```json
{ "kid": "d51ea62a3f49a84cadbd0958e1499ecf" }
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing/invalid field, unsupported `type`, `pubkey` base64 decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | Repo import failed (e.g., malformed pubkey bytes). |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_KEY_IMPORT`.

## Notes

- The `"EXTAID"+pid` descriptor used by ExtAuth pairing ([`/api/auth/ext/validate`](../auth/ext-validate.md)) goes through the same repo import path, so paired authenticators show up in [`/api/keymgmt/list`](list.md) alongside manually-imported peer keys.

## Encedo Manager Usage

```javascript
// build.js:6751
app.api('api/keymgmt/import', 'POST', newKeyData);
```

Manager references: [build.js:6751](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6751), [core2.js:4456](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4456)

## Test Suite Usage

```php
// test_10.php:276, test_8.php:293
http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/import", $ret_val, $keyimport_arg, $token);
```

Test references: [test_10.php:276](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L276), [test_8.php:293](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L293)
