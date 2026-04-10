# POST /api/auth/ext/request

Request authentication via an external authenticator (mobile app). Creates an auth event that is pushed to the paired mobile device.

## Request

```
POST /api/auth/ext/request
Content-Type: application/json
```

**Authentication:** Uses check-in-derived context, not a standard Bearer token.

```json
{
  "epk": "base64_broker_ephemeral_public_key",
  "scope": "requested_access_scope",
  "exp": 3600
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `epk` | String | Yes | Broker ephemeral public key |
| `scope` | String | Yes | Requested access scope |
| `exp` | Number | Yes | Requested token lifetime in seconds |

## Response

**Status:** `200 OK`

```json
{
  "authreq": "jwt_auth_request_string",
  "epk": "base64_device_ephemeral_key"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `authreq` | String | JWT authentication request to forward to mobile |
| `epk` | String | Device ephemeral public key |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 403 | Clock RTC not set |
| 409 | Incorrect internal state |

## Complete Mobile Auth Flow (from [Encedo Manager](https://github.com/encedo/encedo-manager))

```javascript
// build.js:1438-1649
// 1. Check in with device
app.api('api/system/checkin')
// 2. Verify with cloud
.then(data => app.api('https://api.encedo.com/checkin', 'POST', data))
// 3. Post cloud response to device
.then(data => app.api('api/system/checkin', 'POST', data))
// 4. Create cloud notification session
.then(data => app.api('https://api.encedo.com/notify/session'))
// 5. Request auth from device
.then(data => app.api('api/auth/ext/request', 'POST', data))
// 6. Push event to cloud for mobile delivery
.then(data => app.api('https://api.encedo.com/notify/event/new', 'POST', data))
// 7. Poll cloud for mobile's response
// 8. Submit response to /api/auth/ext/token
```

**Source references:** [encedo.js:1316-1327](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1316), [encedo.js:1463-1489](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1463), [build.js:1438-1449](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1438), [build.js:1585-1611](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1585)

## Test Suite Usage

```php
// hem-api-tester/test_6.php:65
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/request", $ret_val, $opt);
```

```php
// hem-api-tester/test_6.php:129
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/request", $ret_val, $opt);
```

**Source references:** [test_6.php:65](https://github.com/encedo/hem-api-tester/blob/main/test_6.php#L65), [test_6.php:129](https://github.com/encedo/hem-api-tester/blob/main/test_6.php#L129)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **NOTE:** [Encedo Manager](https://github.com/encedo/encedo-manager) lists this endpoint in the JWT bypass list ([build.js:214](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L214)), meaning the Bearer header is NOT attached even if a token exists. The auth context comes from the check-in flow rather than a traditional token.
