# POST /api/auth/ext/request

Start an **ExtAuth-issued login**: the caller (typically the Manager, driven by a cloud notification flow) asks the device to emit an `authreq` JWT that every paired authenticator (mobile app) can independently decrypt and countersign. The authenticator that approves replies via the cloud broker; the caller then hands that reply to [`/api/auth/ext/token`](ext-token.md) to receive a Bearer token.

This is the only auth path that produces a Bearer token **without** the caller knowing the user's KDF password — the authenticator approves on the user's behalf.

## Request

```
POST /api/auth/ext/request
Content-Type: application/json
```

**Authentication:** None. This endpoint is deliberately in the **JWT bypass list** — no Bearer is parsed or required. Access control is purely state-based (RTC set, initialised, `fls_state==0`).

**Preconditions (checked in order):**
1. RTC set — otherwise `403`.
2. Initialised — otherwise `409`.
3. `fls_state == 0` — otherwise `409`.

### Body

```json
{
  "epk": "base64_broker_ephemeral_curve25519_public_key",
  "scope": "<requested_scope>",
  "ctx":  "optional_context",
  "note": "optional_note"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `epk` | String | Yes | Broker's ephemeral Curve25519 public key (32 bytes, base64). Does not need to match any paired authenticator — acts as a one-shot transport key. |
| `scope` | String | Yes | Scope the caller wants the resulting token to have. ≤1023 bytes. |
| `ctx` | String | No | Arbitrary 1–64-char context string echoed into the token issued by `/ext/token`. |
| `note` | String | No | Arbitrary 1–128-char note copied into the JWT for the authenticator UI. |

### `scope` rewriting — `keymgmt:use:<kid>`

If the raw `scope` starts with `keymgmt:use:` followed by a valid hex KID, the firmware:

1. Looks up the key in the repo, appends `#<base64({"t":"<type_hex>","l":"<label>"})>` to the scope so the authenticator can show the user *which key* will be used.
2. Shortens the final token lifetime from 60 min to **15 min**.

For any other scope the lifetime is 60 min and no `#`-suffix is added. The hash and everything after is stripped server-side in `/ext/token` before the token is minted.

## Response

**Status:** `200 OK`

```json
{
  "authreq": "<jwt>",
  "epk": "base64_echoed_epk"
}
```

### `authreq` JWT structure

- **Header:** `{"ecdh":"x25519"}`
- **Signed** HS256 with key `ECDH(EIDkey, epk)`.
- **Grants:**
  - `iss` — device `EncedoID` (base64).
  - `aud` — caller's `epk` (base64).
  - `iat`, `exp` — `exp = iat + 60*60` (or `60*15` for rewritten `keymgmt:use:` scopes).
  - `jti` — base64 time-based nonce (SHA-256 sized). This is reused as the per-authenticator GCM key nonce.
  - `ctx`, `note` — echoed if provided.
  - `scope` — **an object**, not a string: each key is base64 of the authenticator KID (32 bytes of descriptor following the `"EXTAID"` prefix), each value is a scheme-tagged ciphertext. Today only scheme `A` (AES-128-CBC + HMAC-SHA256 trailer) is emitted:

    ```
    value = "A" + base64( AES128CBC(plaintext = padded_scope, key = hmac_sha256(jti, ecdh_secret)[0:16], iv = hmac_sha256(jti, ecdh_secret)[16:32])
                          || HMAC-SHA256(plaintext_unpadded, key = hmac_sha256(jti, ecdh_secret)) )
    ```

  The point of this structure: each paired authenticator can derive its own per-request AES key from `ECDH(authenticator_private, EIDkey)` + `jti`, decrypt *its own* entry, and verify the HMAC — other entries in the object are opaque to it.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `epk`/`scope`, `epk` not 32 bytes, or scope >1023 bytes. |
| 403 | RTC not set. |
| 409 | Not initialised, or `fls_state != 0`. |
| 500 | Allocation failure. |

No audit log on this call — audit is written in `/ext/token` on successful token issuance.

## Notes

- The device enumerates up to **8** paired authenticators (`EXT_AUTH_MAX_CLIENTS`). If fewer are paired, the `scope` object simply has fewer entries. Zero paired authenticators still produces a valid-looking `authreq` — the `scope` object is empty and no authenticator can respond; `/ext/token` will return `406`.
- The firmware's `jti` is 32 bytes from `nonce_gen_time_based` (SHA-256 width), even though standard JWT `jti` values are typically shorter. The value is validated back through `nonce_validate_time_based` in `/ext/token`.

## Encedo Manager Usage

Full cloud-round-trip flow:

```javascript
// build.js:1438+
app.api('api/system/checkin')                                 // 1. challenge
  .then(data => app.api('https://api.encedo.com/checkin', 'POST', data))   // 2. broker signs
  .then(data => app.api('api/system/checkin', 'POST', data))  // 3. device accepts
  .then(data => app.api('https://api.encedo.com/notify/session'))
  .then(data => app.api('api/auth/ext/request', 'POST', data))             // 4. authreq
  .then(data => app.api('https://api.encedo.com/notify/event/new', 'POST', data));
// 5. Poll broker for authenticator reply → 6. POST /api/auth/ext/token
```

Manager references: [encedo.js:1316](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1316), [build.js:1438](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1438)

## Test Suite Usage

```php
// test_6.php:65, test_6.php:129
http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/request", $ret_val, $opt);
```

Test references: [test_6.php:65](https://github.com/encedo/hem-api-tester/blob/main/test_6.php#L65), [test_6.php:129](https://github.com/encedo/hem-api-tester/blob/main/test_6.php#L129)
