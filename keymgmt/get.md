# GET /api/keymgmt/get/:kid

Retrieve the public key of an asymmetric key stored in the secure repository.

## Request

```
GET /api/keymgmt/get/:kid
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `keymgmt:get`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `kid` | String | Key ID, 32-character hex string |

## Response

**Status:** `200 OK`

```json
{
  "type": "CURVE25519",
  "pubkey": "base64_encoded_public_key",
  "updated": 1647787070
}
```

| Field | Type | Description |
|-------|------|-------------|
| `type` | String | Key algorithm type |
| `pubkey` | String | Base64-encoded public key |
| `updated` | Number | Last update timestamp |

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

```javascript
// build.js:6145, build.js:6211, core2.js:3850, core2.js:3916
app.api('api/keymgmt/get/' + openedKey.kid).then(function(data) {
    // data.pubkey, data.type used for key details display and sharing
});
```

**Source references:** [build.js:6145](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6145), [build.js:6211](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6211), [core2.js:3850](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3850), [core2.js:3916](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3916)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
