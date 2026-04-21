# POST /api/crypto/pqc/mlkem/decaps

ML-KEM (FIPS 203) decapsulation. Given a decapsulation private key in the repo and a ciphertext from an encapsulator (see [`/api/crypto/pqc/mlkem/encaps`](mlkem-encaps.md)), recover the shared secret.

## Request

```
POST /api/crypto/pqc/mlkem/decaps
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
  "kid": "<32_char_hex_mlkem_kid>",
  "ct":  "<base64_ciphertext>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of an ML-KEM key in the repo. |
| `ct` | String | Yes | Base64 ML-KEM ciphertext from encaps. Decoded length ≤ `WC_ML_KEM_MAX_CIPHER_TEXT_SIZE`. |

## Response

**Status:** `200 OK`

```json
{
  "alg": "MLKEM768",
  "ss":  "<base64_shared_secret>"
}
```

| Field | Description |
|-------|-------------|
| `alg` | One of `MLKEM512`, `MLKEM768`, `MLKEM1024`. |
| `ss` | Base64 shared secret, 32 bytes. Matches the encapsulator's `ss` on success. |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `kid`/`ct`, bad hex/base64, or ciphertext too long. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Key is not ML-KEM, ciphertext not valid for the variant, or decapsulation failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_PQC_MLKEM_DECAPS`. `ss` buffer is zeroed after response.

## Notes

- ML-KEM has implicit rejection: a corrupted / wrong ciphertext will still succeed with a pseudo-random `ss` rather than returning an error. Use paired protocols (e.g. HMAC on a derived key) to detect bad inputs.

## Encedo Manager Usage

Not wired into the Manager UI.

## Test Suite Usage

```php
// test_11.php:867
http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mlkem/decaps", $ret_val, $post_array, $token);
```

Test references: [test_11.php:867](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L867)
