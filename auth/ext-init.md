# POST /api/auth/ext/init

Begin pairing with an **external authenticator** (ExtAuth) — typically the Encedo cloud notification broker or a mobile authenticator app. The external party gets a short record signed with the device's long-lived [EncedoID](../concepts/device-options.md) (`EIDkey`) that it then presents to a secondary channel (e.g. the user scans a QR code on their phone) to prove the device is genuine.

Three endpoints make up the pairing flow:

1. `POST /api/auth/ext/init` — device emits a **request JWT** bound to the broker's ephemeral key.
2. `POST /api/auth/ext/validate` — device accepts the broker's **reply JWT**, imports the paired authenticator's public key into its key repository, and returns a confirmation `code`.
3. `POST /api/auth/ext/mac` — device re-proves its identity to a new ECDH counterparty at any time after pairing.

All three require a Bearer token with `sub="U"` and scope `auth:ext:pair` OR `system:config`.

## Request

```
POST /api/auth/ext/init
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer token required. Scope must be `auth:ext:pair` or `system:config`; `sub` must be `"U"`. Scope matching uses `strstr(scope, X) == scope` (prefix match).

**Preconditions (checked in order):**
1. `fls_state == 0` — otherwise `409`.
2. Device must be initialised — otherwise `409`.
3. Valid JWT — otherwise `401`.

```json
{ "epk": "base64_broker_ephemeral_curve25519_public_key" }
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `epk` | String | Yes | Broker's ephemeral Curve25519 public key (32 bytes, base64). |

## Response

**Status:** `200 OK`

```json
{
  "request": "<jwt>",
  "eid": "base64_encedo_id"
}
```

| Field | Description |
|-------|-------------|
| `request` | JWT signed HS256 over `ECDH(EIDkey, epk)`. Header `{"ecdh":"x25519"}`. Payload: `iss = <eid>`, `aud = <epk>`, `jti = base64(time-based nonce)`, `exp = iat + 86400` (1 day). |
| `eid` | Base64 Curve25519 public key of the device (`Session.EIDkey`). |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON or `epk` not 32 bytes. |
| 401 | Missing or invalid Bearer. |
| 403 | Wrong scope or `sub != "U"`. |
| 409 | `fls_state != 0`, or not initialised. |
| 500 | Allocation failure. |

## Notes

- `request` is **not** a bearer token for this API — it's an opaque JWT the caller forwards out-of-band to the authenticator, which then signs a reply.
- No key material is written on this call; pairing is finalised only on `/api/auth/ext/validate`.

## Encedo Manager Usage

```javascript
// build.js:383+ — pair device with the Encedo cloud broker
app.api('api/system/config')
  .then(data => app.api('https://api.encedo.com/notify/session', 'POST', { eid: data.eid }))
  .then(data => {
      epk_temp = data.epk;
      return app.api('api/auth/ext/init', 'POST', { epk: data.epk });
  })
  .then(data => {
      data.epk = epk_temp;
      return app.api('https://api.encedo.com/notify/register/init', 'POST', data);
  });
```

Manager references: [encedo.js:335](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L335), [build.js:383](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L383)

## Test Suite Usage

```php
// test_3.php:161, test_5.php:76
http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/init", $ret_val, $opt);
```

Test references: [test_3.php:161](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L161), [test_5.php:76](https://github.com/encedo/hem-api-tester/blob/main/test_5.php#L76)
