# POST /api/crypto/ecdh

Run a raw ECDH against a peer public key and return the shared secret directly, or the hash of the shared secret if `alg` is supplied.

Unlike [`/api/keymgmt/derive`](../keymgmt/derive.md), which feeds the ECDH output through HKDF and stores the result as a new repo key, `/api/crypto/ecdh` returns the raw shared bytes to the caller and stores nothing.

## Request

```
POST /api/crypto/ecdh
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** **exact** match `keymgmt:use:<kid_hex>`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Body

```json
{
  "kid":     "<32_char_hex_kid>",
  "ext_kid": "<32_char_hex_peer_kid>",
  "pubkey":  "<base64_peer_public_key>",
  "alg":     "SHA2-256"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. Must be an ECDH-capable private key (Curve25519, Curve448, or a NIST-P / secp256k1 key in `mode: "ECDH"`). |
| `ext_kid` | String | One of | Peer public key already in the repo. |
| `pubkey` | String | One of | Base64 raw peer public key (≤ 66 bytes, fits P-521). |
| `alg` | String | No | Hash applied to the shared secret before return. Omit for raw bytes. |

Exactly one of `ext_kid` / `pubkey` must be present. The peer key's curve must match the `kid` key's curve.

### Hash algorithms

`SHA2-256`, `SHA2-384`, `SHA2-512`, `SHA3-256`, `SHA3-384`, `SHA3-512`. Validated as a ≤8-byte label (`"SHA3-512"` length).

If `alg` is omitted the response contains the bare ECDH output for the underlying curve (32 bytes for X25519/secp256r1/k1, 48 for P-384, 56 for X448, 66 for P-521).

## Response

**Status:** `200 OK`

```json
{ "ecdh": "<base64_shared_secret_or_hash>" }
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing fields, unknown `alg`, or base64/hex decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | ECDH failure (key not ECDH-capable, curve mismatch), hashing failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_ECDH`.

## Notes

- Returning the raw shared secret means the caller sees the full ECDH output. Prefer `/api/keymgmt/derive` (HKDF + fresh stored key) for operational key exchange; use `/api/crypto/ecdh` only when the caller needs the bytes to feed into a custom KDF or protocol.

## Encedo Manager Usage

Not registered in the Manager's `_endpoints` map and not called from Manager UI.

## Test Suite Usage

```php
// test_11.php:571
http_transaction("https", "POST", $cfg_domain, "/api/crypto/ecdh", $ret_val, $post_array, $token);
```

Test references: [test_11.php:571](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L571), [test_11.php:579](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L579)
