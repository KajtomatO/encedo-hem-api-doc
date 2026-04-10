# POST /api/auth/ext/init

Initialize registration of an external authenticator (e.g., Encedo Mobile App). First step of the pairing flow.

## Request

```
POST /api/auth/ext/init
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `auth:ext:pair`

```json
{
  "epk": "base64_broker_ephemeral_public_key"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `epk` | String | Yes | Broker (cloud) ephemeral public key |

## Response

**Status:** `200 OK`

```json
{
  "request": "registration_request_data",
  "eid": "base64_device_public_key"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `request` | String | Registration request data to forward to cloud |
| `eid` | String | Device public key |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Complete Pairing Flow (from [Encedo Manager](https://github.com/encedo/encedo-manager))

```javascript
// build.js:383-396
// 1. Get device config (for eid)
app.api('api/system/config')
// 2. Create cloud session
.then(data => app.api('https://api.encedo.com/notify/session', 'POST', { eid: data.eid }))
// 3. Init pairing on device with cloud's ephemeral key
.then(data => {
    epk_temp = data.epk;
    return app.api('api/auth/ext/init', 'POST', { epk: data.epk });
})
// 4. Register with cloud
.then(data => {
    data.epk = epk_temp;
    return app.api('https://api.encedo.com/notify/register/init', 'POST', data);
});
```

**Source references:** [encedo.js:335-361](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L335), [build.js:383-396](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L383)

## Test Suite Usage

```php
// hem-api-tester/test_3.php:161
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/init", $ret_val, $opt);
```

```php
// hem-api-tester/test_5.php:76
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/init", $ret_val, $opt);
```

**Source references:** [test_3.php:161](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L161), [test_5.php:76](https://github.com/encedo/hem-api-tester/blob/main/test_5.php#L76)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
