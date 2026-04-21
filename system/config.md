# /api/system/config

Read and update the device configuration. The same path behaves very differently depending on whether the device is initialised — in particular, on a fresh (uninitialised) device `POST /api/system/config` accepts an unauthenticated TLS-cert bootstrap and does **nothing else**.

---

## GET /api/system/config

Retrieve the current device configuration (identity, options, storage, network).

### Request

```
GET /api/system/config
Authorization: Bearer <token>
```

**Authentication:** Bearer token required.

**Accepted scopes (any one):**
- `system:config`
- `logger:*` (any `logger:…` scope — the config GET is used during audit-log verification, which needs `eid_sign`)
- `auth:ext:init` (the pairing flow for external authenticators reads the config first)

Invalid token → `401`; scope mismatch → `403` (logged as `LOG_TYPE_FAILED_SCOPE_CHECK`).

### Response

**Status:** `200 OK`

```json
{
  "iat": 1647936471,
  "uts": 1647936500,
  "devid": "hex_device_sn",
  "instanceid": "uuid-hex",
  "eid": "base64_curve25519_public",
  "eid_sign": "base64_ed25519_public",
  "user": "alice",
  "email": "alice@example.com",
  "hostname": "my.ence.do",
  "dnsd": true,
  "trusted_ts": true,
  "trusted_backend": true,
  "allow_keysearch": true,
  "origin": "*",
  "ctx": 1,
  "ip": "192.168.1.10/24",
  "http_option_hsts": true,
  "http_option_dosprot_mode": 0,
  "genuine_id": "atecc608_serial_hex",
  "storage_mode": 163,
  "storage_disk0size": 8388607,
  "storage_capacity": 30777342,
  "spk": "base64_session_public_key",
  "nonce": "base64_time_nonce"
}
```

| Field | Type | When emitted | Description |
|-------|------|--------------|-------------|
| `iat` | Number | Always | Initialisation timestamp (`MainCfg.iat`). |
| `uts` | Number | Always | Last config update timestamp. |
| `devid` | String | Always | 16-char hex of `device_sn` (8 bytes). |
| `instanceid` | String | Always | UUID-format string from `gen_device_uuid()`. |
| `eid` | String | Always | EncedoID — Curve25519 device public key (base64). |
| `eid_sign` | String | Always | Ed25519 audit-log signing public key (base64). |
| `user` | String | Always | User label (`MainCfg.UserIdentity`). |
| `email` | String | Always | User email (`MainCfg.UserEmail`). |
| `hostname` | String | Always | Device hostname (`MainCfg.HostName`). |
| `dnsd` | Boolean | Always | [Device option `dnsd`](../concepts/device-options.md#dnsd--local-dns-daemon) — PPA-only LAN DNS responder. |
| `trusted_ts` | Boolean | Always | [Device option `trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc) — whether the device trusts its own RTC. |
| `trusted_backend` | Boolean | Always | [Device option `trusted_backend`](../concepts/device-options.md#trusted_backend--trust-the-cloud-notification-broker) — whether the cloud broker login path is enabled. |
| `allow_keysearch` | Boolean | Always | [Device option `allow_keysearch`](../concepts/device-options.md#allow_keysearch--enable-unauthenticated-key-search) — unauthenticated EXTAID lookups. |
| `origin` | String | Always | [Device option `origin`](../concepts/device-options.md#origin--cors-origin) — CORS origin header value. |
| `ctx` | Number | Always | Opaque caller-correlation id (`MainCfg.context`). |
| `ip` | String | Always | IPv4 address with CIDR suffix computed from the netmask. |
| `http_option_hsts` | Boolean | Always | [HTTP option `http_option_hsts`](../concepts/device-options.md#http_option_hsts--http-strict-transport-security-header). |
| `http_option_dosprot_mode` | Number | Always | [HTTP option `http_option_dosprot_mode`](../concepts/device-options.md#http_option_dosprot_mode--dos-protection-level) (0–4). |
| `genuine_id` | String | PPA only | ATECC608 hardware serial number (hex). |
| `storage_mode` | Number | PPA only | [`storage_mode`](../concepts/device-options.md#storage_mode--storage-layout--crypto-mode) bitfield. |
| `storage_disk0size` | Number | PPA only | [`storage_disk0size`](../concepts/device-options.md#storage_disk0size--plaintext-partition-size) in sectors. Note the value is `MainCfg.StoragePlainSize - 1` (the `-1` is a firmware-historical off-by-one exposed on the wire). |
| `storage_capacity` | Number | PPA only | Total usable microSD sectors across both slots minus 2. |
| `spk` | String | Always | Per-boot session public key (base64). Same value as returned by `GET /api/auth/token`. |
| `nonce` | String | Always | Fresh time-based nonce (base64) — callers that need to provide a nonce in follow-up writes (e.g. new `userkey`) can use this one. |

---

## POST /api/system/config

Two very different behaviours depending on device state.

### Pre-init behaviour — TLS bootstrap

If the device is **not** initialised, the handler **ignores** everything except a `tls` object in the body. It:

1. Saves the raw body to `/tmp/tls.json` (PPA only) or passes it to the TLS web-server start task (EPA).
2. Starts the HTTPS listener if it isn't already running.
3. Returns `200` if the body was JSON containing a `tls` key, `406` if it wasn't, `400` if the body wasn't JSON.

No Bearer token, no role check, no scope check. No other config fields are applied pre-init — `POST /api/auth/init` is the only way to commit the initial config.

### Post-init behaviour — normal config update

```
POST /api/system/config
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required.
- Scope must start with `system:config`.
- `sub` claim must be `"U"` (User-password token) or `"M"` (Master-password token). Any other `sub` → `403`.
- If a firmware/UI install is in progress → `409`.

### Request body

Any combination of the fields below. Any field the firmware doesn't recognise is silently ignored (but `updated` is still `true` only if something actually changed). If **any** recognised field fails validation, the whole request returns `400` with **no partial application** (the new config is only committed if all validators pass).

| Field | Type | Effect |
|-------|------|--------|
| `wipeout` | Boolean | If `true`, runs `system_full_wipeout()` and reboots. Response is `200` sent immediately before reboot. `false` currently returns `406`. Requires `sub ∈ {M, U}`. |
| `user` | String | Updates `MainCfg.UserIdentity` (validated via `isvalid_label`). |
| `email` | String | Updates `MainCfg.UserEmail`. |
| `origin` | String | Updates [`origin`](../concepts/device-options.md#origin--cors-origin). |
| `trusted_ts` | Boolean | Updates [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc). |
| `trusted_backend` | Boolean | Updates [`trusted_backend`](../concepts/device-options.md#trusted_backend--trust-the-cloud-notification-broker). |
| `allow_keysearch` | Boolean | Updates [`allow_keysearch`](../concepts/device-options.md#allow_keysearch--enable-unauthenticated-key-search). |
| `dnsd` | Boolean | PPA only. Updates [`dnsd`](../concepts/device-options.md#dnsd--local-dns-daemon). |
| `http_option_hsts` | Boolean | Updates [`http_option_hsts`](../concepts/device-options.md#http_option_hsts--http-strict-transport-security-header). |
| `http_option_dosprot_mode` | Number (0–4) | Updates [`http_option_dosprot_mode`](../concepts/device-options.md#http_option_dosprot_mode--dos-protection-level). |
| `storage_mode` | Number | PPA only. Updates [`storage_mode`](../concepts/device-options.md#storage_mode--storage-layout--crypto-mode). Changing disk1 crypto mode triggers a disk reformat. |
| `storage_disk0size` | Number | PPA only. Updates [`storage_disk0size`](../concepts/device-options.md#storage_disk0size--plaintext-partition-size). Values larger than the card minus 128 KB are clamped. Triggers a disk reformat. |
| `ctx` | Number | Updates `MainCfg.context`. |
| `userkey` + `userkey_hmac` + `userkey_nonce` | Strings (all three required) | Rotate the User public key. `userkey` is the new Curve25519 public key (base64). `userkey_nonce` must be a valid time-based nonce from a recent GET on this device. `userkey_hmac` is `HMAC-SHA256(userkey_nonce, ECDH(Session.session_key, userkey))` — proves the caller holds the matching private key. |
| `tls` | Object | Install a new TLS certificate (and optionally private key). Triggers `reboot_required`. See below. |
| `gen_csr` | Boolean `true` | On init-aged devices, regenerate a CSR from the stored TLS private key using the current hostname and email. The CSR is returned in the response. |

### `tls` sub-object

Two variants:

1. **Cert + key bundle.** `tls.emp` (base64 DER P-256 pubkey), `tls.key` (`<base64-der-csr>.<base64-der-ciphertext>`; the key is AES-256-CBC-encrypted under the shared secret from ATECC608 ECDH with `emp`), `tls.crt` (base64 DER certificate).
2. **Cert only.** `tls.crt` alone — replaces the stored certificate, leaving the existing private key in place.

Successfully importing either sets `tls_imported` and responds with `"reboot_required": true`.

### Response

**Status:** `200 OK` on success (even if nothing changed); `400` on validator failure.

```json
{
  "updated": true,
  "reboot_required": true,
  "csr": "PEM-ish CSR",
  "genuine": "atecc608_attestation_token"
}
```

| Field | When emitted | Description |
|-------|--------------|-------------|
| `updated` | Always | `true` if the new config CRC differs from the current one or TLS was re-imported. |
| `reboot_required` | Only when a TLS cert/key was imported | Tells the Manager to offer a reboot. |
| `csr` | Only when `gen_csr: true` succeeded | PEM-ish CSR text. |
| `genuine` | Always when `csr` is emitted | ATECC608 attestation token over the CSR. On EPA this is the precomputed `Session.epa_genuine`. |

On storage-geometry change (disk1 crypto mode or disk0 size), the device starts `StartTaskFormatDisks(1)` after committing the new config — a foreground disk reformat. Poll `GET /api/system/status` for `format` progress.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON or any field failed validation. |
| 401 | Missing/invalid Bearer. |
| 403 | Scope or role mismatch. |
| 406 | `wipeout: false` (no-op sentinel). |
| 409 | Firmware/UI install in progress. |
| 500 | Allocation failure. |

## Notes

- Empty POST bodies (or bodies whose fields are all unrecognised) return `200` with `"updated": false` and **no audit log entry**. An audit entry (`LOG_TYPE_CONFIG_UPDATED`) is only written when the CRC actually changes or TLS is imported.
- `userkey` rotation does not clear any audit-log or repo state; it only changes the public key used by `POST /api/auth/token` (for `iss` matching).

## Encedo Manager Usage

```javascript
// encedo.js:374 etc
app.api('api/system/config');
app.api('api/system/config', 'POST', { user: "alice", email: "a@b.c" });
```

Manager references: [encedo.js:374](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L374), [build.js:417](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L417), [build.js:3260](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L3260), [core2.js:2950](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2950)

## Test Suite Usage

```php
// test_0.php:100
http_transaction("https", "GET", $cfg_domain, "/api/system/config", $ret_val, $post_dummy, $token);
// test_0.php:107
http_transaction("https", "POST", $cfg_domain, "/api/system/config", $ret_val, $post_data, $token);
```

Test references: [test_0.php:100](https://github.com/encedo/hem-api-tester/blob/main/test_0.php#L100), [test_1.php:118](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L118), [test_7.php:81](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L81)
