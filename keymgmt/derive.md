# POST /api/keymgmt/derive

Derive a fresh key by running ECDH against a second public key, then feeding the shared secret through HKDF-SHA256 with an algorithm-specific context label. The output is stored as a new key in the repo and returned by `kid`.

The "source" side (`kid` in the request) must reference an ECDH-capable asymmetric key that the device holds *privately* (typically one created by `/api/keymgmt/create` with a Curve25519 or NIST-P type + `mode: "ECDH"`). The "peer" side is either another key already in the repo (`ext_kid`) or a raw base64 public key (`pubkey`).

## Request

```
POST /api/keymgmt/derive
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** prefix-match `keymgmt:derive` **or** `keymgmt:gen`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Body

```json
{
  "kid":     "<32_char_hex_source_ecdh_kid>",
  "label":   "<human_label>",
  "type":    "<derived_key_type>",
  "ext_kid": "<32_char_hex_peer_ecdh_kid>",
  "pubkey":  "<base64_peer_public_key>",
  "mode":    "ECDH" | "ECDH,ExDSA" | "ExDSA",
  "descr":   "<base64_opaque_blob>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex. Must point at an ECDH-capable private key on the device. |
| `label` | String | Yes | Label for the derived key (`isvalid_label` ≤32 bytes). |
| `type` | String | Yes | Type of the **derived** key — see below. |
| `ext_kid` | String | One of | 32-char hex of a public key already in the repo. |
| `pubkey` | String | `ext_kid`/`pubkey` | Base64 raw public key. Max 66 bytes (fits P-521). |
| `mode` | String | No | NIST-P ECC derived keys only. |
| `descr` | String | No | Base64 metadata blob. |

Exactly one of `ext_kid` / `pubkey` must be present; providing both returns `400`.

### Derivation pipeline

1. `shared = ECDH(source_kid_private, peer_public)` — via `CRYPTO_DeriveKey` (curve is determined by the source key's algorithm).
2. `ikm = HKDF-SHA256(shared, salt=∅, info="encedo-<type>")`, expand to the derived key's native length:

   | `type` | `info` string | Output length |
   |--------|---------------|---------------|
   | `SECP256R1` | `encedo-secp256r1` | 32 |
   | `SECP384R1` | `encedo-secp384r1` | 48 |
   | `SECP521R1` | `encedo-secp521r1` | 66 |
   | `SECP256K1` | `encedo-secp256k1` | 32 |
   | `CURVE25519` | `encedo-curve25519` | 32 |
   | `CURVE448` | `encedo-curve448` | 56 |
   | `ED25519` | `encedo-ed25519` | 32 |
   | `ED448` | `encedo-ed448` | 57 |
   | `SHA2-256` | `encedo-sha256` | 32 |
   | `SHA2-384` | `encedo-sha384` | 48 |
   | `SHA2-512` | `encedo-sha512` | 64 |
   | `SHA3-256` | `encedo-sha3-256` | 32 |
   | `SHA3-384` | `encedo-sha3-384` | 48 |
   | `SHA3-512` | `encedo-sha3-512` | 64 |
   | `AES128` | `encedo-aes128` | 16 |
   | `AES192` | `encedo-aes192` | 24 |
   | `AES256` | `encedo-aes256` | 32 |

3. The derived bytes are passed to `REPO_GenKey_*` with `seed=ikm`, producing a deterministic key from the shared secret.

PQC types (`MLKEM*`, `MLDSA*`) are **not** supported by derive.

## Response

**Status:** `200 OK`

```json
{ "kid": "bd0958e1499ecfd51ea62a3f49a84cad" }
```

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing required field, both/neither of `ext_kid`/`pubkey` set, unsupported `type`. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | ECDH failed (source key not ECDH-capable, peer key wrong curve), HKDF failure, or repo write failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_KEY_DERIVE`.

## Notes

- The derived key is **deterministic** in the source private key, the peer public key, and the algorithm family (because the `info` label is fixed). Two parties that share their public keys and agree on a `type` will end up with the same derived key — useful for pair-wise symmetric channels without explicit key exchange.
- Because `info` is hard-coded per type, deriving two different types against the same peer produces independent keys.

## Encedo Manager Usage

Not invoked directly by the Manager UI; registered in the endpoints/scopes map for third-party API consumers ([build.js:9233](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9233)).

## Test Suite Usage

```php
// test_10.php:175
http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/derive", $ret_val, $keyderive_arg, $token);
```

Test references: [test_10.php:175](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L175)
