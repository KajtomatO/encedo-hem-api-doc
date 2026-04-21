# /api/auth/token

User authentication via passphrase. Two-step process: fetch a challenge, then submit a signed JWT to receive a scoped Bearer token.

---

## GET /api/auth/token

Fetch authentication challenge.

### Request

```
GET /api/auth/token
```

**Authentication:** Not required.

**Preconditions (checked in order, before challenge is generated):**
1. If the device option [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc) is **true**, the RTC must be set — otherwise `403`. (`trusted_ts` makes the device trust its own RTC and refuse to issue time-bound tokens when the clock is unset.)
2. The device must be initialised (i.e. `POST /api/auth/init` has completed) — otherwise `409`.

### Response

**Status:** `200 OK`

```json
{
  "exp": 1647871504,
  "spk": "base64_session_public_key",
  "jti": "base64_jwt_id_nonce",
  "lbl": "My device",
  "eid": "base64_encedo_id"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `exp` | Number | Challenge expiration — 60 seconds after issuance. The `jti`/`spk` pair is only valid within this window. |
| `spk` | String | Session public key (base64, X25519) — the device's ephemeral ECDH key for this challenge. Caller uses this as `aud` and for ECDH. |
| `jti` | String | Time-based nonce (base64, 36 bytes = 4-byte timestamp || SHA-256 HMAC). The device validates this value on POST. |
| `lbl` | String | User label (username/identity) set during initialisation. |
| `eid` | String | EncedoID — device's long-lived Curve25519 public key (base64). Callers typically use this as salt for password-based KDFs. |

### Error Codes

| Code | Description |
|------|-------------|
| 403 | RTC not set (only when [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc) is enabled) |
| 409 | Device not initialised |
| 500 | Internal error generating challenge |

---

## POST /api/auth/token

Submit authentication data and receive a scoped Bearer token.

### Request

```
POST /api/auth/token
Content-Type: application/json
```

```json
{
  "auth": "signed_jwt_string"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `auth` | String | Yes | JWT signed with the X25519 ECDH shared secret (see below). |

### Authentication JWT

The JWT signing key is the **shared secret derived by X25519 ECDH** between the caller's private key and the device's `spk` from the GET challenge. The device derives the same secret on its side from its ephemeral `spk` private key and the caller's public key carried in the `iss` claim.

**JWT header:**
```json
{ "ecdh": "x25519" }
```
The device reads the `ecdh` field to decide whether to run ECDH on the `iss` grant before HMAC verification (`jwt-wolfssl.c`). Standard header fields (`alg`, `typ`) are ignored; the algorithm is always HS256 over the ECDH-derived secret. See [DISCREPANCIES-ENCEDO-MANAGER.md](../discrepancies/DISCREPANCIES-ENCEDO-MANAGER.md) for header variants.

**JWT payload:**
```json
{
  "jti": "<challenge_jti>",
  "aud": "<spk from GET response>",
  "exp": <challenge_exp>,
  "iat": <current_timestamp>,
  "iss": "<caller_public_key_base64>",
  "scope": "keymgmt:list",
  "ctx": "optional_audit_tag"
}
```

| Claim | Required | Description |
|-------|----------|-------------|
| `jti` | Yes | Echo the `jti` from the GET challenge. The device recomputes the time-based HMAC and rejects on mismatch. |
| `aud` | Yes | Echo the `spk` from the GET challenge (device's session public key). |
| `exp` | Yes | Expiration of the eJWT. May be the challenge `exp` or a smaller value. If `exp < iat`, the device substitutes a 30-second default for the issued bearer's lifetime. |
| `iat` | Yes | Issue timestamp. If [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc) is disabled and this value is > 2021-09, the device may use it to prime its RTC. |
| `iss` | Yes | Caller's Curve25519 public key (base64). Must equal either the `UserKey` ("U") or `MasterKey` ("M") registered at init — any other value returns `401`. |
| `scope` | No | Scope to attach to the issued Bearer token. If empty/omitted, the issued token has **no scope** and can only be used with endpoints that accept unscoped or any-token access. |
| `ctx` | No | Opaque free-form string, ≤64 bytes. Copied verbatim into the issued Bearer token's `ctx` claim. Has no other side-effect. |

### Response

**Status:** `200 OK`

```json
{
  "token": "<jwt_bearer_token>"
}
```

The returned Bearer JWT's payload contains:

| Claim | Description |
|-------|-------------|
| `sub` | `"U"` for User-key-derived token, `"M"` for Master-key-derived token. |
| `scope` | Requested scope (omitted if none was requested). |
| `iat` | Issue timestamp. |
| `exp` | Expiration timestamp (caller-requested or `iat + AUTH_TOKEN_LIFETIME` default). |
| `jti` | Per-token time-based nonce. |
| `ctx` | Echoed from the request, if provided. |

The Bearer token is signed with the same ECDH-derived session key, so it is only verifiable by the same device during the same session (i.e. until the device reboots or a new challenge is issued).

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `auth`, or missing/invalid `cfg` object. |
| 401 | JWT decode failed, JWT validation failed, `jti` nonce invalid/expired, or `iss` did not match the User/Master key. Subject to anti-bruteforce delay (see below). |
| 403 | RTC not set (only when [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc) is enabled). |
| 409 | Device not initialised. |
| 500 | Internal error. |

### Anti-Bruteforce Delay

Every failing attempt increments an in-RAM counter (`Session.auth_fails_local`). Once the counter exceeds the firmware-compiled threshold, the device holds the response for a multiplier of the base delay before returning the status code. Repeated failures therefore slow the caller down and rate-limit online password guessing. The counter decays after a lockdown window with no further failures.

Callers should implement sensible retry spacing; retrying immediately on `401` simply tightens the rate limit.

---

## Authentication Flow — Example Usage

The device itself is agnostic to how the caller materialises the keypair whose public half is stored as `UserKey`/`MasterKey`. In practice there are two reference implementations, each using a different password-based KDF. A device only authenticates callers whose public key matches the one registered during `POST /api/auth/init`, so the KDF must match the one used at init time.

### Example 1: Encedo Manager (Argon2)

```javascript
// encedo-manager: build.js:851-929, encedo.js:447-827
app.api('api/auth/token').then(function(init) {
    // 1. Derive 32-byte secret from password using Argon2
    argon2.hash({ pass: password, salt: FromBase64(init.eid), time: 10, mem: 8192, hashLen: 32 })
    .then(function(hash) {
        // 2. Generate X25519 keypair from the derived secret
        var keys = nacl.box.keyPair.fromSecretKey(hash.hash);
        // 3. ECDH against the device's session public key
        var sharedSecret = nacl.scalarMult(keys.secretKey, FromBase64(init.spk));
        // 4. Sign eJWT with the shared secret
        var jwt = jwt_generate_hs256(
            { ecdh: "x25519" },
            { jti: init.jti, aud: init.spk, exp: init.exp, iat: now, iss: keys.publicKey, scope },
            sharedSecret);
        // 5. Submit
        return app.api('api/auth/token', 'POST', { auth: jwt });
    })
    .then(function(reply) { self.jwt_token = reply.token; });
});
```

Manager references: [encedo.js:447-827](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L447), [build.js:490-929](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L490), [core2.js:4528-4568](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4528)

### Example 2: HEM API test suite (PBKDF2)

```php
// hem-api-tester: libs/lib.php — password-to-secret KDF
$secret = hash_pbkdf2('sha256', $password, base64_decode($eid), 600000, 32, true);
// Then identical X25519 / ECDH / HS256 JWT flow as above.
```

The test suite also sets a fuller JWT header:
```json
{ "alg": "HS256", "typ": "JWT", "ecdh": "x25519" }
```
The device ignores `alg`/`typ` and only branches on the `ecdh` field — both header forms are accepted.

Test suite references: [test_3.php:71](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L71), [test_3.php:97](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L97), [libs/lib.php:305](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php#L305), [libs/lib.php:337](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php#L337)

> **Note:** The two reference implementations cannot interoperate on the same device unless the same KDF was used during `POST /api/auth/init` — the resulting public keys differ. See [DISCREPANCIES-ENCEDO-MANAGER.md](../discrepancies/DISCREPANCIES-ENCEDO-MANAGER.md) and [DISCREPANCIES-HEM-TEST.md](../discrepancies/DISCREPANCIES-HEM-TEST.md).

## Notes

- Tokens are cached by scope in the Manager (`self.tokens[scope]`) with their own expiration; the flow above is repeated for each required scope.
- The issued Bearer is attached to subsequent requests via the `Authorization: Bearer <token>` header.
- Unauthenticated endpoints that skip the Bearer header even when a token is cached: `api/system/status`, `api/system/version`, `api/system/checkin`, `api/auth/ext/token`, `api/auth/ext/request`, `api/auth/token`, `api/auth/init`.
