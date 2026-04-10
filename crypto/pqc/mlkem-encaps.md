# POST /api/crypto/pqc/mlkem/encaps

Perform ML-KEM (FIPS 203) encapsulation -- generate a shared secret and ciphertext from an encapsulation key.

## Request

```
POST /api/crypto/pqc/mlkem/encaps
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}`

**Allowed users:** User, ExtAuth

```json
{
  "kid": "32char_hex_key_id"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) -- must reference an MLKEM key |

## Response

**Status:** `200 OK`

```json
{
  "ct": "base64_encoded_ciphertext",
  "ss": "base64_encoded_shared_secret",
  "alg": "MLKEM768"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ct` | String | Base64-encoded ciphertext (to send to the other party) |
| `ss` | String | Base64-encoded shared secret (use for symmetric encryption) |
| `alg` | String | Algorithm used (MLKEM512, MLKEM768, or MLKEM1024) |

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
| MLKEM512 | FIPS 203 ML-KEM-512 |
| MLKEM768 | FIPS 203 ML-KEM-768 |
| MLKEM1024 | FIPS 203 ML-KEM-1024 |

The algorithm is determined by the key type -- create the key with the desired MLKEM variant via `POST /api/keymgmt/create`.

## Encedo Manager Usage

**Not present in [Encedo Manager](https://github.com/encedo/encedo-manager).** This is a newer firmware feature not yet integrated into the Manager UI.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:858
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mlkem/encaps", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:858](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L858)

## Official Docs vs Encedo Manager

> **NOTE:** Documented in official docs under `/post-quantum-cryptography/ml-kem`. Not referenced anywhere in the Manager JavaScript.
