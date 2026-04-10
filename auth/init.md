# /api/auth/init

Device initialization (personalization). Two-step process: get challenge, then submit initialization data.

---

## GET /api/auth/init

Get initialization challenge data from an unpersonalized device.

### Request

```
GET /api/auth/init
```

**Authentication:** Not required

### Response

**Status:** `200 OK`

```json
{
  "exp": 1647936471,
  "spk": "base64_server_public_key",
  "jti": "base64_jwt_id",
  "genuine": "base64_attestation_data",
  "eid": "base64_encedo_id"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `exp` | Number | Expiration timestamp for the challenge |
| `spk` | String | Server public key (base64) -- used for ECDH |
| `jti` | String | JWT ID / nonce (base64) |
| `genuine` | String | Attestation data (base64) |
| `eid` | String | EncedoID -- device public key (base64), used as Argon2 salt |

### Error Codes

| Code | Description |
|------|-------------|
| 403 | Clock RTC not set |
| 406 | Operation failed |
| 409 | Incorrect internal state (device already initialized) |

---

## POST /api/auth/init

Submit the initialization JWT with master key, user key, and device configuration.

### Request

```
POST /api/auth/init
Content-Type: application/json
```

```json
{
  "init": "signed_jwt_string"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `init` | String | Yes | JWT signed with ECDH shared secret containing the init config |

### JWT Payload Structure

The `init` JWT contains:

```json
{
  "jti": "challenge_jti",
  "aud": "server_public_key",
  "exp": "challenge_exp",
  "iat": "current_timestamp",
  "iss": "master_public_key",
  "cfg": {
    "masterkey": "base64_master_public_key",
    "userkey": "base64_user_public_key",
    "user": "username",
    "email": "user@example.com",
    "hostname": "my.ence.do",
    "ip": "192.168.1.1",
    "storage_mode": 1,
    "storage_disk0size": 1024,
    "dnsd": true,
    "trusted_ts": true,
    "trusted_backend": true,
    "allow_keysearch": true,
    "origin": "*",
    "gen_csr": true
  }
}
```

JWT header: `{ "ecdh": "x25519" }`
JWT signing: HMAC-SHA256 with ECDH shared secret derived from `masterSecretKey` x `spk`.

### Response

**Status:** `200 OK`

```json
{
  "instanceid": "e7aafede-76b1-c7c8-784b-c963967fe307",
  "token": "jwt_access_token",
  "genuine": "attestation_data"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `instanceid` | String | Newly created instance UUID |
| `token` | String | JWT access token for post-init operations |
| `genuine` | String | Attestation data |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Initialization JWT validation failed |
| 403 | Clock RTC not set |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:533-638
// Step 1: Get challenge
app.api('api/auth/init').then(function(encedo_authinit) {
    // Step 2: Generate keys
    // - Argon2(password, eid) -> userSecret -> NaCl keypair
    // - BIP39 mnemonic -> masterSecret -> NaCl keypair
    // - ECDH: masterSecretKey x spk -> sharedSecret
    // Step 3: Sign JWT and POST
    var jwt = self.jwt_generate_hs256(header, dataFinal, sharedSecret);
    xhr.open('POST', self.encedo_url + '/api/auth/init');
    xhr.send(JSON.stringify({ init: jwt }));
});
```

**Source references:** [encedo.js:488-638](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L488), [build.js:533-758](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L533)

## Test Suite Usage

```php
// hem-api-tester/test_2.php:82
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/auth/init", $ret_val);
```

```php
// hem-api-tester/test_2.php:128
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/auth/init", $ret_val, $opt);
```

**Source references:** [test_2.php:82](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L82), [test_2.php:128](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L128), [test_2.php:245](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L245)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **NOTE:** The official docs describe `init` as an opaque "initialization structure" string. [Encedo Manager](https://github.com/encedo/encedo-manager) reveals it is a JWT signed with `HS256` using an X25519 ECDH-derived shared secret, containing a `cfg` object with all personalization parameters.

> **CRITICAL DISCREPANCY (KDF):** The Manager uses **Argon2** for password-to-key derivation during init; the [HEM API test suite](https://github.com/encedo/hem-api-tester) uses **PBKDF2-SHA256** (600,000 iterations). See [token.md](token.md) for full details.

> **JWT header discrepancy:** The Manager sets `{ "ecdh": "x25519" }`. The test suite sets `{ "alg": "HS256", "typ": "JWT", "ecdh": "x25519" }`. Both are accepted by the device.
