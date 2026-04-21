# POST /api/crypto/pqc/mlkem/encaps

ML-KEM (FIPS 203) encapsulation. Given an encapsulation key already in the repo (`MLKEM512` / `MLKEM768` / `MLKEM1024`), produce a fresh shared secret `ss` and a ciphertext `ct` that the holder of the corresponding decapsulation key can use to recover `ss`.

## Request

```
POST /api/crypto/pqc/mlkem/encaps
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** **exact** match `keymgmt:use:<kid_hex>`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Body

```json
{ "kid": "<32_char_hex_mlkem_kid>" }
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of an ML-KEM key in the repo. |

## Response

**Status:** `200 OK`

```json
{
  "alg": "MLKEM768",
  "ss":  "<base64_shared_secret>",
  "ct":  "<base64_ciphertext>"
}
```

| Field | Description |
|-------|-------------|
| `alg` | One of `MLKEM512`, `MLKEM768`, `MLKEM1024` — determined by the key type on the device. |
| `ss` | Base64 shared secret, `WC_ML_KEM_SS_SZ` (32) bytes before encoding. |
| `ct` | Base64 ML-KEM ciphertext; length depends on `alg` (768/1088/1568 bytes). |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `kid`, or bad hex. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Key is not ML-KEM or encapsulation failed. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_PQC_MLKEM_ENCAPS`. Both `ss` and `ct` buffers are zeroed after response.

## Notes

- The ML-KEM variant is fixed at key creation — create the key via [`/api/keymgmt/create`](../../keymgmt/create.md) with the desired `MLKEM512/768/1024` type, then reuse its `kid` on encaps.
- For a full KEM session: encaps on the sender side, send `ct` over the wire, decaps with [`/api/crypto/pqc/mlkem/decaps`](mlkem-decaps.md) on the receiver. Both sides end up with the same `ss`.

## Encedo Manager Usage

Not wired into the Manager UI.

## Test Suite Usage

```php
// test_11.php:858
http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mlkem/encaps", $ret_val, $post_array, $token);
```

Test references: [test_11.php:858](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L858)
