# POST /api/crypto/pqc/mldsa/verify

Verify an ML-DSA (FIPS 204) signature against a message using a public key in the repo.

## Request

```
POST /api/crypto/pqc/mldsa/verify
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
  "kid":  "<32_char_hex_mldsa_kid>",
  "msg":  "<base64_message>",
  "sign": "<base64_signature>",
  "ctx":  "<base64_context>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | 32-char hex of an ML-DSA key. |
| `msg` | String | Yes | Base64 plaintext. Decoded length ≤ `MAX_API_POST_CRYTO_MSG_LEN`. |
| `sign` | String | Yes | Base64 signature. Decoded length ≤ `DILITHIUM_ML_DSA_87_SIG_SIZE` (4627). |
| `ctx` | String | No | Base64 signing context. Decoded length ≤ 255 bytes (matches the `ctx` limit on sign). |

## Response

**Status:** `200 OK` — no body on successful verification.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `kid`/`msg`/`sign`, or decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope not exact `keymgmt:use:<kid>` or `sub == "M"`. |
| 406 | Signature invalid for the given `msg`+`ctx`+key, or wrong key type. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_CRYPTO_PQC_MLDSA_VERIFY`.

## Encedo Manager Usage

Not wired into the Manager UI.

## Test Suite Usage

```php
// test_11.php:930
http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mldsa/verify", $ret_val, $post_array, $token);
```

Test references: [test_11.php:930](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L930)
