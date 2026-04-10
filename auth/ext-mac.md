# POST /api/auth/ext/mac

Obtain MAC data to authenticate the device on the broker (cloud) and retrieve a list of paired external authenticators.

## Request

```
POST /api/auth/ext/mac
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
| `epk` | String | Yes | Broker ephemeral public key |

## Response

**Status:** `200 OK`

```json
{
  "nonce": "base64_nonce",
  "mac": "base64_mac",
  "eid": "base64_device_public_key"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `nonce` | String | Nonce value |
| `mac` | String | MAC for cloud authentication |
| `eid` | String | Device public key |

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
// Used in paired() and unpair() flows
// build.js:427, build.js:463
app.api('api/auth/ext/mac', 'POST', { epk: data.epk })
.then(function(data) {
    data.epk = epk_temp;
    // Forward to cloud to list/delete subscribers
    return app.api('https://api.encedo.com/notify/subscribers/list', 'POST', data);
});
```

**Source references:** [encedo.js:384-426](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L384), [build.js:427-469](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L427)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
