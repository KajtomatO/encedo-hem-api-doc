# POST /api/auth/ext/token

Third and final step of the ExtAuth login flow: submit the authenticator's **signed reply** and receive a Bearer token. The authenticator signed its reply over `ECDH(authenticator_private, EIDkey_public)`; the device validates the signature with its own `EIDkey` private key, recovers the scope the authenticator approved, and issues a short-lived JWT.

## Request

```
POST /api/auth/ext/token
Content-Type: application/json
```

**Authentication:** Not required. Security comes from the ECDH signature of `authreply`.

**Preconditions:**
1. RTC set — otherwise `403`.
2. Initialised — otherwise `409`.
3. `fls_state == 0` — otherwise `409`.

Anti-bruteforce: every failure adds `auth_delay_response_remote` delay.

### Body

```json
{ "authreply": "<jwt>" }
```

### `authreply` JWT shape (produced by the paired authenticator)

- **Header:** `{"ecdh":"x25519"}`
- **Signature key:** HS256 with `ECDH(authenticator_private, EIDkey_public)`; device verifies with `EIDkey` private.
- **Grants:**
  - `jti` — base64 of the same 32-byte time-based nonce the device emitted in `authreq.jti`.
  - `iss` — authenticator's Curve25519 public key (must match a key stored in the repo under descriptor `"EXTAID" + base64_decode(pid)`).
  - `pid` — base64 of the 32-byte descriptor suffix that locates the authenticator's key (same value that was assigned at `/ext/validate`).
  - `scope` — base64-encoded AES-CBC ciphertext with scheme prefix `"A"`, covering the scope approved by the authenticator. Key = `HMAC-SHA256(jti, ECDH(authenticator_private, EIDkey))`. Trailing HMAC-SHA256 of the plaintext (unpadded) authenticates the payload.
  - `ctx` — optional, copied from `authreq.ctx` if present.

Only scheme `"A"` is accepted today — other scheme prefixes return `406`.

## Response

**Status:** `200 OK`

```json
{ "token": "<bearer_jwt>" }
```

### Issued Bearer

- **Header:** `{"alg":"HS256","typ":"JWT"}`
- **Signature:** HS256 with `Session.session_key` (same per-boot key as `/api/auth/token` tokens).
- **Grants:**
  - `sub` — base64 of the ExtAuth `kid` (i.e. the paired authenticator identity), **not** `"U"` / `"M"`. The token represents "this authenticator approved" rather than a user password login.
  - `scope` — the decrypted scope from `authreply.scope`. If the authenticator was asked for a `keymgmt:use:<kid>` scope, any `#`-suffix metadata is stripped before the token is issued.
  - `iat` — current time.
  - `exp` — the `exp` claim copied from the original `authreq` (15 min for `keymgmt:use:` scopes, 60 min otherwise).
  - `jti` — a fresh time-based nonce, **not** the `jti` from `authreply`.
  - `ctx` — copied through from `authreply` if present.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON or missing `authreply`. |
| 401 | JWT decode/validate failed, or `jti` invalid. |
| 403 | RTC not set. |
| 406 | `iss` doesn't match any paired authenticator; scheme not `"A"`; AES-CBC decrypt or HMAC trailer mismatch. |
| 409 | Not initialised, or `fls_state != 0`. |
| 500 | Allocation failure. |

Audit log on success: `LOG_TYPE_AUTH_SUCCESS_EXTERNAL` with `sub = base64(kid)` and the decrypted scope.

## Notes

- Every failing path calls `auth_delay_response_remote(start_ts, 1)` — the handler deliberately delays the response so a bruteforce caller experiences increasing latency.
- The ExtAuth path does **not** increment the local/global password-fail counters (`auth_fails_*`) that `/api/auth/token` maintains; it has a separate failure backoff.
- The resulting token is indistinguishable on the wire from a password-login token — scope enforcement on downstream endpoints is identical.

## Encedo Manager Usage

```javascript
// build.js:1649
self.api('api/auth/ext/token', 'POST', { authreply: data.authreply })
    .then(function(token_data) { self.jwt_token = token_data.token; });
```

Manager references: [encedo.js:1527](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1527), [build.js:1649](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1649)

## Test Suite Usage

```php
// test_6.php:94
http_transaction("https", "POST", $cfg_domain, "/api/auth/ext/token", $ret_val, $opt);
```

Test references: [test_6.php:94](https://github.com/encedo/hem-api-tester/blob/main/test_6.php#L94)
