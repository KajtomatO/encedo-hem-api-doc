# /api/auth/init

Device initialisation (one-shot personalisation). Two-step process: fetch a challenge, then submit a signed init JWT containing the `cfg` block that commits the initial configuration to flash.

An initialised device returns `406` from both verbs — reinit requires a full wipeout first (`POST /api/system/config {"wipeout":true}` or the DIAG wipe).

---

## GET /api/auth/init

Fetch an init challenge.

### Request

```
GET /api/auth/init
```

**Authentication:** Not required.

**Preconditions (checked in order):**
1. RTC must be set — otherwise `403`. Unlike `/api/auth/token`, init **always** requires a set RTC (regardless of [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc)).
2. Device must **not** already be initialised — otherwise `406`.
3. `fls_state` must be `0` — otherwise `409`.

### Response

**Status:** `200 OK`

```json
{
  "exp": 1647871504,
  "spk": "base64_session_public_key",
  "jti": "base64_jwt_id_nonce",
  "genuine": "base64_atecc608_attestation_token",
  "eid": "base64_encedo_id"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `exp` | Number | Challenge expiration — 60 seconds after issuance. |
| `spk` | String | Session public key (base64, X25519) — the device's ephemeral ECDH key. |
| `jti` | String | Base64 36-byte time-based nonce. |
| `genuine` | String | ATECC608 attestation token (PPA) or `Session.epa_genuine` (EPA). Lets the caller verify they are talking to a real Encedo device before trusting it with the master key. |
| `eid` | String | EncedoID — device's long-lived Curve25519 public key (base64). Used as salt for the caller's password KDF. |

### Error Codes

| Code | Description |
|------|-------------|
| 403 | RTC not set. |
| 406 | Device already initialised. |
| 409 | `fls_state != 0` (self-test failure). |
| 500 | Allocation failure. |

---

## POST /api/auth/init

Commit the initial configuration.

### Request

```
POST /api/auth/init
Content-Type: application/json
```

```json
{ "init": "<signed_jwt>" }
```

**Authentication:** Not required. Security comes from the ECDH signature on the JWT.

**Preconditions (checked in order):**
1. RTC set; otherwise `403`.
2. Not already initialised; otherwise `406`.
3. `fls_state == 0`; otherwise `409`.

### Init JWT

Signed HS256 with the ECDH shared secret between the caller's private key and the device's `spk` from the GET response — same construction as `/api/auth/token`.

**Header:**
```json
{ "ecdh": "x25519" }
```

**Payload:**
```json
{
  "jti": "<challenge_jti>",
  "aud": "<spk>",
  "exp": <challenge_exp>,
  "iat": <current_timestamp>,
  "iss": "<caller_public_key_base64>",
  "cfg": {
    "masterkey": "base64_curve25519_public",
    "userkey":   "base64_curve25519_public",
    "user":      "Alice",
    "email":     "alice@example.com",
    "hostname":  "my.ence.do",
    "ip":        "192.168.1.10/24",
    "storage_mode":       83,
    "storage_disk0size":  8388608,
    "dnsd":            true,
    "trusted_ts":      true,
    "trusted_backend": true,
    "allow_keysearch": true,
    "origin":          "*",
    "gen_csr":         true,
    "ctx":             0
  }
}
```

### `cfg` fields

The device tracks each recognised field as a bit in a completeness mask and refuses the whole request (`400`) unless **all 14 bits** are set.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user` | String | Yes | Initial `MainCfg.UserIdentity`. |
| `masterkey` | String | Yes | Curve25519 public key of the master-password persona. Master-key tokens can reconfigure/wipeout the device. |
| `userkey` | String | Yes | Curve25519 public key of the user-password persona. Can be rotated later via `POST /api/system/config`. |
| `hostname` | String | Yes | Device hostname. |
| `ip` | String | Yes (PPA) / auto (EPA) | `A.B.C.D/prefix`. EPA has no network stack configurable here — the bit is auto-set. |
| `origin` | String | Yes | CORS origin ([`origin`](../concepts/device-options.md#origin--cors-origin)). |
| `storage_mode` | Number | Yes (PPA) / auto (EPA) | [`storage_mode`](../concepts/device-options.md#storage_mode--storage-layout--crypto-mode). |
| `storage_disk0size` | Number | Yes (PPA) / auto (EPA) | [`storage_disk0size`](../concepts/device-options.md#storage_disk0size--plaintext-partition-size). |
| `trusted_ts` | Boolean | Yes | [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc). |
| `trusted_backend` | Boolean | Yes | [`trusted_backend`](../concepts/device-options.md#trusted_backend--trust-the-cloud-notification-broker). |
| `allow_keysearch` | Boolean | Yes | [`allow_keysearch`](../concepts/device-options.md#allow_keysearch--enable-unauthenticated-key-search). |
| `dnsd` | Boolean | Yes (PPA) / auto (EPA) | [`dnsd`](../concepts/device-options.md#dnsd--local-dns-daemon). |
| `email` | String | Yes | `MainCfg.UserEmail`. |
| `gen_csr` | Boolean | Yes | Whether to generate a device TLS CSR during init (see below). |
| `ctx` | Number | No | Opaque context id. Defaults to `0` if omitted. Does **not** participate in the completeness mask. |

Validation rules per field are the same as in `POST /api/system/config` — labels go through `isvalid_label`, numbers are range-clamped where applicable, `storage_disk0size` is clamped to `sd_card_sector_count - 128KB`.

### Response

**Status:** `200 OK`

```json
{
  "reboot_required": true,
  "instanceid": "uuid-hex",
  "token": "<bearer_jwt>",
  "csr": "PEM_CSR",
  "genuine": "attestation_token"
}
```

| Field | When emitted | Description |
|-------|--------------|-------------|
| `reboot_required` | When hostname, IP, storage_mode (disk1 crypto), or disk0 size changed from defaults | Tells the caller to reboot. |
| `instanceid` | Always | New device UUID (HKDF over DeviceKey). |
| `token` | Always | Bearer JWT with `sub="U"` and scope `system:config`. Use this to finalise setup (import TLS cert, rotate userkey, etc.). |
| `csr` | Only when `gen_csr: true` succeeded | PEM-encoded TLS CSR generated from a new on-device P-256 key; the key is stored in the repo with descriptor `DNS:<hostname>`. |
| `genuine` | Always | Fresh ATECC608 attestation token (PPA) or `Session.epa_genuine` (EPA). |

Side effects on success:
- Committed config is flashed (`flash_write_user_signature`).
- Key repository is initialised with `StorageSecrets`.
- Audit log entry `LOG_TYPE_DEVICE_INITED` with `sub="M"`.
- On PPA, `StartTaskFormatDisks(0)` is kicked off after the response — the first-boot format runs in the background.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `init`, missing `cfg` object, or any of the 14 mandatory `cfg` fields missing/invalid. |
| 401 | JWT decode/validate failed, or `jti` invalid. |
| 403 | RTC not set. |
| 406 | Device already initialised, or flash write failed. |
| 409 | `fls_state != 0`. |
| 500 | Allocation failure. |

## Notes

- The firmware is **KDF-agnostic**: any caller whose public key matches `masterkey` or `userkey` at login time is accepted. Both reference implementations (Manager, test suite) use the same KDF: PBKDF2-HMAC-SHA256, 600 000 iterations, 32-byte output, salt = the `eid` string as UTF-8 bytes (not base64-decoded). See [auth/token.md](token.md).
- There is no built-in way to change `masterkey` after init. Rotating it requires a full wipeout + re-init.

## Encedo Manager Usage

```javascript
// build.js:655+ (initFinal)
app.api('api/auth/init').then(function(challenge) {
    // 1. PBKDF2-SHA256(userPassword, salt=eid, 600000 iter, 32 bytes) → userkey keypair
    // 2. BIP39 mnemonic → masterSecret → masterkey keypair
    // 3. ECDH(masterSecretKey, spk) → sharedSecret
    // 4. jwt_generate_hs256({ecdh:"x25519"}, { jti, aud, exp, iat, iss, cfg }, sharedSecret)
    return app.api('api/auth/init', 'POST', { init: jwt });
});
```

Manager references: [build.js:655](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L655) (`initFinal`; KDF call at [build.js:678](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L678)), called from [build.js:537](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L537)

## Test Suite Usage

```php
// test_2.php:82 (GET)
http_transaction("https", "GET", $cfg_domain, "/api/auth/init", $ret_val);
// test_2.php:128 (POST)
http_transaction("https", "POST", $cfg_domain, "/api/auth/init", $ret_val, $opt);
```

Test references: [test_2.php:82](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L82), [test_2.php:128](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L128), [test_2.php:245](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L245)
