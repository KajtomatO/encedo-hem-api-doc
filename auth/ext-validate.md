# POST /api/auth/ext/validate

Second step of external-authenticator pairing. The caller forwards the `request` JWT from [`/api/auth/ext/init`](ext-init.md) to an external authenticator (mobile app, broker, …), collects a signed reply, and uploads it here. On success the paired authenticator's public key is imported into the device key repository under descriptor prefix `"EXTAID"` and an ExtAuth slot is consumed.

## Request

```
POST /api/auth/ext/validate
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer token required. Scope `auth:ext:pair` or `system:config`; `sub="U"`. Same access check as `/api/auth/ext/init`.

**Preconditions:**
1. `fls_state == 0` — otherwise `409`.
2. Device initialised — otherwise `409`.
3. Valid JWT — otherwise `401`.

```json
{
  "pid": "base64_pairing_id",
  "reply": "<reply_jwt>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pid` | String | Yes | Pairing ID chosen by the caller. Base64-decoded and prepended with `"EXTAID"` to form the repo descriptor. |
| `reply` | String | Yes | JWT emitted by the external authenticator, signed HS256 over `ECDH(authenticator_private, EIDkey_public)`. |

### `reply` JWT shape

- **Header:** `{"ecdh":"x25519"}`
- **Signature key:** HS256 with key = `ECDH(authenticator_private_key, EIDkey_public)`. The device validates using its own EIDkey **private** key.
- **Required grants:**
  - `jti` — base64 time-based nonce (time-validated against firmware clock).
  - `iss` — authenticator's Curve25519 public key (base64). Imported into repo as-is.
  - `label` — free-form label the mobile app chose for itself (string).
  - `epk` — a Curve25519 public key the authenticator will use as its ECDH-confirmation input. Mixed with `EIDkey` to derive a shared secret, which HMACs the caller-visible `reply` JWT to produce the returned `code`.

## Response

**Status:** `200 OK`

```json
{
  "kid": "hex_key_id",
  "code": "base64_confirmation_code"
}
```

| Field | Description |
|-------|-------------|
| `kid` | Hex-encoded key id under which the authenticator's public key was stored in the device repo. Use this in future `POST /api/auth/ext/request` if scope-bound to one specific authenticator. |
| `code` | HMAC-SHA256 of the full `reply` JWT keyed by `ECDH(EIDkey, epk)`, base64-encoded. The caller forwards this to the authenticator so the authenticator can verify that the device received and accepted its reply. |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON; missing `pid`/`reply`; `epk` in reply not 32 bytes. |
| 401 | Missing/invalid Bearer, or `reply` JWT decode/validate/jti-nonce fails. |
| 403 | Wrong scope or `sub != "U"`. |
| 406 | Repo import failed (likely slot table full — see notes). |
| 409 | `fls_state != 0`, or not initialised. |
| 500 | Allocation failure. |

Audit log on success: `LOG_TYPE_AUTH_PAIRED_EXTERNAL` with `sub="U"` and the new `kid`.

## Notes

- The firmware caps ExtAuth clients at **8** slots (`EXT_AUTH_MAX_CLIENTS`). Additional pairings fail at `REPO_ImportKey` with `406`. Remove an entry via the normal keymgmt delete flow before re-pairing.
- The repo descriptor is exactly `"EXTAID" + base64_decode(pid)`, so `pid` collisions overwrite/conflict — the caller is responsible for choosing a unique pid.
- `request`/`reply` JWTs are one-time: the `jti` nonce is time-based and single-use; a replayed `reply` produces `401`.

## Encedo Manager Usage

```javascript
// build.js:4887, core2.js:2631
app.api('api/auth/ext/validate', 'POST', { pid, reply });
```

Manager references: [build.js:4887](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L4887), [core2.js:2631](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2631)

## Test Suite Usage

```php
// test_5.php:120
http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/validate", $ret_val, $opt);
```

Test references: [test_5.php:120](https://github.com/encedo/hem-api-tester/blob/main/test_5.php#L120)
