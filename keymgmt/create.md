# POST /api/keymgmt/create

Generate a fresh key and store it in the on-device key repository. The key material is generated from the TRNG; the caller only supplies metadata (`label`, `descr`) and the algorithm family.

## Request

```
POST /api/keymgmt/create
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** prefix-match `keymgmt:gen`.
- **Subject:** `sub != "M"` (master-key tokens can't touch key material).

**Preconditions (checked in order):**
1. `fls_state == 0` — otherwise `409`.
2. No software install in progress — otherwise `409`.
3. Valid JWT — otherwise `401`.
4. TLS connection — otherwise `418`.
5. Initialised — otherwise `409`.

### Body

```json
{
  "type":  "<algorithm>",
  "label": "<human_label>",
  "mode":  "ECDH" | "ECDH,ExDSA" | "ExDSA",
  "descr": "<base64_opaque_blob>"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `type` | String | Yes | One of the supported types below. |
| `label` | String | Yes | `isvalid_label(label, STOREDKEY_LABEL_MAX_LENGTH)` — printable, ≤32 bytes. |
| `mode` | String | No | Only meaningful for NIST-P ECC keys. Sets bit flags on the stored key. Must be one of the three literals. |
| `descr` | String | No | Base64 blob. Decoded length ≤ `STOREDKEY_DESCR_MAX_LENGTH`. Opaque to the firmware; used for client-side tagging and prefix search in [`/api/keymgmt/search`](search.md). |

### Supported `type` values

| Family | Values |
|--------|--------|
| NIST ECC | `SECP256R1`, `SECP384R1`, `SECP521R1`, `SECP256K1` |
| Edwards/Montgomery | `CURVE25519`, `CURVE448`, `ED25519`, `ED448` |
| HMAC | `SHA2-256`, `SHA2-384`, `SHA2-512`, `SHA3-256`, `SHA3-384`, `SHA3-512` |
| AES | `AES128`, `AES192`, `AES256` |
| PQC KEM | `MLKEM512`, `MLKEM768`, `MLKEM1024` |
| PQC DSA | `MLDSA44`, `MLDSA65`, `MLDSA87` |

## Response

**Status:** `200 OK`

```json
{ "kid": "09bd0958e1499ecfd51ea62a3f49a84c" }
```

| Field | Description |
|-------|-------------|
| `kid` | 32-char hex, 16-byte key id. Use this in [get](get.md) / [update](update.md) / [delete](delete.md) / the `keymgmt:use:<kid>` crypto scopes. |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing/invalid `label`/`type`, unsupported `type`, invalid `mode`, invalid `descr` base64. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | Key-repo write failed (out of space, etc.). |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_KEY_GENERATION` — `LOG_RESULT_OK` with the new `kid` on success; `LOG_RESULT_ERROR`/`LOG_RESULT_FAILED` on validation or repo failure.

## Notes

- `mode` is silently ignored for types where it doesn't apply (all non-NIST-P ECC). Only NIST-P ECC keys carry the `STOREDKEY_TYPE_ECDH`/`STOREDKEY_TYPE_ECDSA` role flags at creation time.
- The repo stores label + descr as metadata, not as part of the crypto material. You can rename a key later via [`/api/keymgmt/update`](update.md) without rotating the key.
- Naming conventions are literal strings. AES uses no dash (`AES128`, not `AES-128`); HMAC uses the raw hash family name (`SHA2-256`, not `HMAC-SHA2-256`).

## Encedo Manager Usage

```javascript
// build.js:6662
app.api('api/keymgmt/create', 'POST', newKeyData).then(data => { /* data.kid */ });
```

Manager references: [build.js:6662](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6662), [core2.js:4367](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4367)

## Test Suite Usage

```php
// test_10.php:115
http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/create", $ret_val, $keycreate_arg, $token);
```

Test references: [test_10.php:115](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L115)
