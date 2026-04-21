# GET /api/keymgmt/get/{kid}

Fetch the **public half** (or DER bytes for imported certs / wrapped DER private keys) of a key in the repository, together with metadata. Symmetric key material (AES/HMAC) is never exported — only the algorithm type string is returned.

## Request

```
GET /api/keymgmt/get/{kid}
Authorization: Bearer <token>
```

**Authentication:** Bearer required, TLS required.

- **Scope:** any of
  - prefix-match `keymgmt:get`, **or**
  - prefix-match `keymgmt:gen` (key creation scope — create→read pattern), **or**
  - **exact** match `keymgmt:use:<kid-hex>` (a token minted for *this specific* key via [`/api/auth/ext/request`](../auth/ext-request.md)).
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Path parameters

| Parameter | Description |
|-----------|-------------|
| `kid` | 32-char hex. |

## Response

**Status:** `200 OK`

Shape depends on the key's type.

### Asymmetric keys (ECC, Edwards/Montgomery, ML-KEM, ML-DSA)

```json
{
  "type":    "CURVE25519" | "ED25519" | "SECP256R1" | … | "MLKEM1024" | "MLDSA87",
  "pubkey":  "<base64_raw_public_key>",
  "updated": 1647787070,
  "descr":   "<base64_opaque_blob>"
}
```

Key bytes use the algorithm-native little-endian / X9.63 encoding (same wire format the `import`/`create`/`ecdh` endpoints accept).

### Certificates (imported via `import` with type `CERT`)

```json
{
  "type":    "CERT",
  "der":     "<base64_der_cert>",
  "updated": 1647787070,
  "descr":   "<base64_opaque_blob>"
}
```

### Wrapped private DER blobs

```json
{
  "type":    "DER_PKEY",
  "der":     "<base64_der_private_key>",
  "updated": 1647787070,
  "descr":   "<base64_opaque_blob>"
}
```

### Symmetric keys (HMAC / AES)

```json
{
  "type":    "HMAC-SHA2-256" | "AES-128" | …,
  "updated": 1647787070,
  "descr":   "<base64_opaque_blob>"
}
```

No key material is returned. The exact type string comes from `__repo_keytype2string()` inside the firmware.

| Field | Always emitted? | Notes |
|-------|-----------------|-------|
| `type` | Yes | Algorithm name. |
| `pubkey` | Only for asymmetric, non-DER types | Raw public bytes, base64. |
| `der` | Only for `CERT` / `DER_PKEY` | DER-encoded, base64. |
| `updated` | Yes | Unix time of last update (metadata change or creation). |
| `descr` | Only if non-empty | Base64 blob set by create/import/update. |
| `label` | **Never** | Despite the label being stored in the repo, the GET handler deliberately omits it (the `cJSON_AddStringToObject(json, "label", label)` line is commented out in the firmware). |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | `kid` missing or not exactly 32 hex chars. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | `kid` not found, or asymmetric key export failed. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

## Notes

- The `keymgmt:use:<kid>` exact-scope path is how ExtAuth-issued tokens reference individual keys. Issuing a token scoped to `keymgmt:use:abc…` gives the token holder read access to that one key and nothing else.
- `descr` is opaque to the firmware. Encedo Manager uses it for per-key tagging and the `^EXTAID`-prefix scheme that marks paired authenticator entries (see [`/api/keymgmt/search`](search.md)).

## Encedo Manager Usage

```javascript
// build.js:6145
app.api('api/keymgmt/get/' + kid).then(data => { /* data.pubkey, data.type */ });
```

Manager references: [build.js:6145](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6145), [core2.js:3850](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3850)

## Test Suite Usage

Exercised indirectly — most tests call list/search rather than get-by-kid.
