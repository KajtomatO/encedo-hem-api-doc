# POST /api/auth/ext/mac

Prove device identity to an already-known counterparty. The device derives `ECDH(EIDkey, epk)`, HMACs a fresh nonce with that secret, and returns both. The counterparty (typically the Encedo notification broker) runs the same ECDH with its own key pair and verifies the HMAC — establishing a live liveness+identity check without any on-device state change.

Unlike `/api/auth/ext/init` + `/validate`, this endpoint does **not** pair a new authenticator. It's used by already-paired cloud flows (subscriber list / unpair / notify session bootstrap) to re-authenticate the device.

## Request

```
POST /api/auth/ext/mac
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required. Scope `auth:ext:pair` or `system:config`; `sub="U"`.

**Preconditions:** `fls_state == 0`, initialised, valid JWT — otherwise `409` / `401`.

```json
{ "epk": "base64_counterparty_curve25519_public_key" }
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `epk` | String | Yes | Counterparty's Curve25519 public key (32 bytes, base64). |

## Response

**Status:** `200 OK`

```json
{
  "nonce": "base64_32_byte_nonce",
  "mac":   "base64_hmac_sha256",
  "eid":   "base64_encedo_id"
}
```

| Field | Description |
|-------|-------------|
| `nonce` | 32-byte time-based nonce (base64). |
| `mac` | `HMAC-SHA256(nonce, key=ECDH(EIDkey, epk))` (base64). |
| `eid` | Device's long-lived Curve25519 public key ([EncedoID](../concepts/device-options.md)). |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON or `epk` not 32 bytes. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub != "U"`. |
| 409 | `fls_state != 0`, or not initialised. |
| 500 | Allocation failure. |

## Notes

- No audit-log entry is written — this is a liveness probe, not a state change.
- The nonce is generated via the time-based nonce pool (same mechanism as `/api/auth/token`'s `jti`), but the firmware does **not** validate/consume it on the return path; the counterparty is responsible for rejecting replay. In practice, replays are bounded by the counterparty's own session lifetime.

## Encedo Manager Usage

```javascript
// build.js:427 — list subscribers, build.js:463 — unpair
app.api('api/auth/ext/mac', 'POST', { epk: data.epk })
  .then(function(data) {
      data.epk = epk_temp;
      return app.api('https://api.encedo.com/notify/subscribers/list', 'POST', data);
  });
```

Manager references: [encedo.js:384](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L384), [build.js:427](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L427)

## Test Suite Usage

Not exercised directly in `hem-api-tester`.
