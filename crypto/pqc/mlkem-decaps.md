# POST /api/crypto/pqc/mlkem/decaps

Perform ML-KEM (FIPS 203) decapsulation -- extract a shared secret from ciphertext using the decapsulation key.

## Request

```
POST /api/crypto/pqc/mlkem/decaps
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:use:{kid}`

**Allowed users:** User, ExtAuth

```json
{
  "kid": "32char_hex_key_id",
  "ct": "base64_encoded_ciphertext"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) -- must reference an MLKEM key |
| `ct` | String | Yes | Base64-encoded ciphertext from encapsulation |

## Response

**Status:** `200 OK`

```json
{
  "ss": "base64_encoded_shared_secret"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ss` | String | Base64-encoded shared secret (matches the encapsulator's `ss`) |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

**Not present in [Encedo Manager](https://github.com/encedo/encedo-manager).** See [mlkem-encaps.md](mlkem-encaps.md) for details.

## Test Suite Usage

```php
// hem-api-tester/test_11.php:867
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/crypto/pqc/mlkem/decaps", $ret_val, $post_array, $token);
```

**Source references:** [test_11.php:867](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L867)

## Official Docs vs Encedo Manager

> **NOTE:** Documented in official docs. Not in [Encedo Manager](https://github.com/encedo/encedo-manager).
