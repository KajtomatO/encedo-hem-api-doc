# POST /api/auth/ext/validate

Validate/finalize external authenticator registration. Uploads the reply data from the external authenticator to complete pairing.

## Request

```
POST /api/auth/ext/validate
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `auth:ext:pair`

```json
{
  "pid": "unique_pairing_id",
  "reply": "reply_data_from_external_authenticator"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pid` | String | Yes | Unique Pairing ID |
| `reply` | String | Yes | Reply data from external authenticator |

## Response

**Status:** `200 OK`

```json
{
  "kid": "key_id_hex_32chars",
  "code": "confirmation_code"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `kid` | String | Key ID of the paired authenticator |
| `code` | String | Confirmation code for user verification |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:4887, core2.js:2631
app.api('api/auth/ext/validate', 'POST', data);
```

**Source references:** [build.js:4887](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L4887), [core2.js:2631](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2631)

## Test Suite Usage

```php
// hem-api-tester/test_5.php:120
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/validate", $ret_val, $opt);
```

**Source references:** [test_5.php:120](https://github.com/encedo/hem-api-tester/blob/main/test_5.php#L120)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
