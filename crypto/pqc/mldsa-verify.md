# POST /api/crypto/pqc/mldsa/verify

Verify an ML-DSA (FIPS 204) post-quantum digital signature.

## Request

```
POST /api/crypto/pqc/mldsa/verify
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}`

**Allowed users:** User, ExtAuth

```json
{
  "kid": "32char_hex_key_id",
  "msg": "base64_encoded_message",
  "sign": "base64_encoded_signature",
  "ctx": "base64_encoded_context"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) -- must reference an MLDSA key |
| `msg` | String | Yes | Base64-encoded message (max 2048 bytes) |
| `sign` | String | Yes | Base64-encoded signature to verify |
| `ctx` | String | No | Base64-encoded context data (max 64 bytes) |

## Response

**Status:** `200 OK` -- Signature verified successfully

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed (verification failed) |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

**Not present in [Encedo Manager](https://github.com/encedo/encedo-manager).** See [mldsa-sign.md](mldsa-sign.md) for details.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:930
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mldsa/verify", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:930](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L930)

## Official Docs vs Encedo Manager

> **NOTE:** Documented in official docs. Not in [Encedo Manager](https://github.com/encedo/encedo-manager). Note the `ctx` max size differs between sign (255 bytes) and verify (64 bytes) per the official docs.
