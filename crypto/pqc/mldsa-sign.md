# POST /api/crypto/pqc/mldsa/sign

Produce an ML-DSA (FIPS 204) post-quantum signature over a message using a private key already in the repo (`MLDSA44` / `MLDSA65` / `MLDSA87`).

## Request

```
POST /api/crypto/pqc/mldsa/sign
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
  "kid": "<32_char_hex_mldsa_kid>",
  "msg": "<base64_message>",
  "ctx": "<base64_context>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of an ML-DSA key in the repo. |
| `msg` | String | Yes | Base64 plaintext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. |
| `ctx` | String | No | Base64 signing context. Decoded length ≤ 255 bytes (FIPS 204 cap). |

## Response

**Status:** `200 OK`

```json
{
  "alg":  "MLDSA65",
  "sign": "<base64_signature>"
}
```

| Field | Description |
|-------|-------------|
| `alg` | One of `MLDSA44`, `MLDSA65`, `MLDSA87` — determined by the key on device. |
| `sign` | Base64 ML-DSA signature. Length depends on variant (2420 / 3309 / 4627 bytes). |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `kid`/`msg`, or base64/hex decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Key not ML-DSA or signing failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_PQC_MLDSA_SIGN`. Signature buffer is zeroed after response.

## Notes

- The ML-DSA variant is fixed at key creation. Create the key via [`/api/keymgmt/create`](../../keymgmt/create.md).

## Encedo Manager Usage

Not wired into the Manager UI. Key creation paths in Manager cover `MLDSA*` types but signing is not invoked.

## Test Suite Usage

```php
// test_11.php:922
http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mldsa/sign", $ret_val, $post_array, $token);
```

Test references: [test_11.php:922](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L922)
