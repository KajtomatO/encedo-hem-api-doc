# POST /api/auth/ext/token

Submit the mobile authenticator's reply to obtain a Bearer token. Final step of the external authentication flow.

## Request

```
POST /api/auth/ext/token
Content-Type: application/json
```

**Authentication:** Not required (this IS the authentication endpoint)

```json
{
  "authreply": "auth_reply_data_from_mobile"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `authreply` | String | Yes | Authentication reply data from the external authenticator |

## Response

**Status:** `200 OK`

```json
{
  "token": "jwt_bearer_token"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `token` | String | JWT access token (same format as `/api/auth/token` response) |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | `authreply` not authenticated |
| 403 | Clock RTC not set |
| 406 | Issuer not found |
| 409 | Conflict |

## Encedo Manager Usage

```javascript
// build.js:1649
self.api('api/auth/ext/token', 'POST', { authreply: data.authreply })
.then(function(token_data) {
    self.jwt_token = token_data.token;
});
```

**Source references:** [encedo.js:1527](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1527), [build.js:1649](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1649)

## Test Suite Usage

```php
// hem-api-tester/test_6.php:94
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/token", $ret_val, $opt);
```

**Source references:** [test_6.php:94](https://github.com/encedo/hem-api-tester/blob/main/test_6.php#L94)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
