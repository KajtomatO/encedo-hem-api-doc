# POST /api/crypto/pqc/mldsa/sign

Generate an ML-DSA (FIPS 204) post-quantum digital signature.

## Request

```
POST /api/crypto/pqc/mldsa/sign
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}`

**Allowed users:** User, ExtAuth

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_message",
  "ctx": "base64_encoded_context"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) -- must reference an MLDSA key |
| `msg` | String | Yes | Base64-encoded message to sign (max 2048 bytes) |
| `ctx` | String | No | Base64-encoded context data (max 255 bytes) |

## Response

**Status:** `200 OK`

```json
{
  "sign": "base64_encoded_signature",
  "alg": "MLDSA65"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `sign` | String | Base64-encoded ML-DSA signature |
| `alg` | String | Algorithm used (MLDSA44, MLDSA65, or MLDSA87) |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

### Supported Algorithms

| Algorithm | Standard |
|-----------|----------|
| MLDSA44 | FIPS 204 ML-DSA-44 |
| MLDSA65 | FIPS 204 ML-DSA-65 |
| MLDSA87 | FIPS 204 ML-DSA-87 |

The algorithm is determined by the key type -- create the key with the desired MLDSA variant via `POST /api/keymgmt/create`.

## Encedo Manager Usage

**Not present in [Encedo Manager](https://github.com/encedo/encedo-manager).** This is a newer firmware feature. Keys of type MLDSA44/65/87 can be created via `/api/keymgmt/create` (which is in [Encedo Manager](https://github.com/encedo/encedo-manager)), but the signing endpoint is not yet integrated into the Manager UI.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:922
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mldsa/sign", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:922](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L922)

## Official Docs vs Encedo Manager

> **NOTE:** Documented in official docs under `/post-quantum-cryptography/ml-dsa`. Not referenced anywhere in the Manager JavaScript. The endpoint path uses `/api/crypto/pqc/` prefix, different from classical crypto endpoints which use `/api/crypto/`.
