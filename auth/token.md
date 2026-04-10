# /api/auth/token

User authentication via passphrase. Two-step process: get challenge, then submit authentication JWT.

---

## GET /api/auth/token

Get authentication challenge.

### Request

```
GET /api/auth/token
```

**Authentication:** Not required

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
| `exp` | Number | Challenge expiration timestamp |
| `spk` | String | Session public key (base64) -- for ECDH |
| `jti` | String | JWT ID / nonce (base64) |
| `lbl` | String | Device label |
| `eid` | String | EncedoID (base64) -- used as Argon2 salt |

### Error Codes

| Code | Description |
|------|-------------|
| 403 | Clock RTC not set |
| 409 | Incorrect internal state |

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
| `auth` | String | Yes | JWT signed with ECDH shared secret |

### JWT Payload Structure

```json
{
  "jti": "challenge_jti",
  "aud": "server_public_key",
  "exp": "challenge_exp",
  "iat": "current_timestamp",
  "iss": "user_public_key",
  "scope": "keymgmt:list"
}
```

JWT header: `{ "ecdh": "x25519" }`
JWT signing: HMAC-SHA256 with ECDH shared secret derived from `userSecretKey` x `spk`.

The `scope` field determines what operations the returned token permits. Default is `"master"` if not specified.

### Response

**Status:** `200 OK`

```json
{
  "token": "jwt_bearer_token"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `token` | String | JWT access token. Contains `scope`, `sub`, `iat`, `exp`, `jti` claims. |

The returned token's `sub` field indicates user type: `"U"` (User), `"M"` (Master), etc.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | User not authenticated (wrong password / key mismatch) |
| 403 | Clock RTC not set |
| 409 | Incorrect internal state |

## Authentication Flow (from [Encedo Manager](https://github.com/encedo/encedo-manager))

```javascript
// build.js:851-929
// 1. Get challenge
app.api('api/auth/token').then(function(init) {
    // 2. Derive key from password
    argon2.hash({ pass: password, salt: FromBase64(init.eid), time: 10, mem: 8192, hashLen: 32 })
    .then(function(hash) {
        // 3. Generate keypair
        var keys = nacl.box.keyPair.fromSecretKey(hash.hash);
        // 4. Compute shared secret
        var sharedSecret = nacl.scalarMult(keys.secretKey, FromBase64(init.spk));
        // 5. Build and sign JWT
        var jwt = jwt_generate_hs256(header, { jti, aud: init.spk, exp, iat, iss: keys.publicKey, scope }, sharedSecret);
        // 6. Submit
        app.api('api/auth/token', 'POST', { auth: jwt })
        .then(function(reply) {
            self.jwt_token = reply.token;  // Store for Bearer auth
        });
    });
});
```

**Source references:** [encedo.js:447-827](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L447), [build.js:490-929](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L490), [build.js:6823-6863](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6823), [core2.js:4528-4568](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4528)

## Notes

- Tokens are cached by scope in `self.tokens[scope]` with their expiration time.
- The token is automatically attached to subsequent requests via the `Authorization: Bearer` header.
- Endpoints listed in the bypass list (`api/system/status`, `api/system/version`, `api/system/checkin`, `api/auth/ext/token`, `api/auth/ext/request`, `api/auth/token`) skip the Bearer header even if a token exists.

## Test Suite Usage

```php
// hem-api-tester/test_3.php:71
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/auth/token", $ret_val);
```

```php
// hem-api-tester/test_3.php:97
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/token", $ret_val, $opt);
```

**Source references:** [test_3.php:71](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L71), [test_3.php:97](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L97), [libs/lib.php:305](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php#L305), [libs/lib.php:337](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php#L337)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **NOTE:** The official docs describe `auth` as an opaque string. [Encedo Manager](https://github.com/encedo/encedo-manager) reveals the complete JWT structure with ECDH key exchange and Argon2 KDF.

> **CRITICAL DISCREPANCY (KDF):** The Manager uses **Argon2** (time=10, mem=8192, hashLen=32) for password key derivation, while the [HEM API test suite](https://github.com/encedo/hem-api-tester) ([libs/lib.php](https://github.com/encedo/hem-api-tester/blob/main/libs/lib.php)) uses **PBKDF2-SHA256** (600,000 iterations, 32 bytes). Both use the device `eid` as salt and produce a 32-byte secret fed into X25519. This means the Manager and the test suite derive **different keys from the same password**, so they cannot authenticate to the same device instance. The device firmware likely supports both KDF methods, or one of the implementations is outdated.

> **JWT header discrepancy:** The Manager sets `{ "ecdh": "x25519" }`. The test suite sets `{ "alg": "HS256", "typ": "JWT", "ecdh": "x25519" }`. The device likely accepts both forms.
