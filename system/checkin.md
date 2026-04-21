# /api/system/checkin

Device-to-cloud check-in handshake. The GET builds a signed challenge describing the running firmware (signature, bootloader signature, UI hash); the POST accepts a counter-signed response that may deliver a fresh TLS cert, prime the RTC, advertise available upgrades, or trigger a remote lock/wipe/brick action.

The cloud broker (`api.encedo.com/checkin`) sits between these two calls; the device never talks to the internet directly.

---

## GET /api/system/checkin

Generate a signed check-in challenge describing this device and its firmware.

### Request

```
GET /api/system/checkin
```

**Authentication:** Not required.

### Response

**Status:** `200 OK`

```json
{ "check": "<jwt>" }
```

The `check` value is a JWT with header `{"ecdh":"x25519"}` and the following claims. It is signed HS256 with the **ECDH shared secret** derived between this boot's ephemeral `Session.session_key` and the firmware's application public key (`MyFooter.ApplicationPublicKey`). Only the holder of the firmware's application private key — i.e. the publisher's backend — can verify/respond to it.

| Claim | When emitted | Description |
|-------|--------------|-------------|
| `jti` | Always | Base64 of a 36-byte time-based nonce. Validated on POST. |
| `iss` | Always | Base64 of `Session.sessid` — the device's ephemeral session public key for this power-up. |
| `qid` | Always | Base64 of `Session.sessdevid` — per-boot HMAC of DeviceID, opaque outside the publisher. |
| `aud` | Always | Base64 of the firmware's application public key (identifies the running firmware publisher). |
| `fws` | Always | Base64 of the firmware signature (proves running image). |
| `bls` | Only when bootloader footer is present and its publisher matches the firmware's | Base64 of the bootloader signature. |
| `uis` | PPA only | Base64 of the installed Manager UI hash (`Session.ui_ver_hash`). |
| `csn` | Only if a TLS cert is loaded **and** `HostName` ends with the default TLD | Base64 of the TLS x509 serial number. Lets the broker detect a stale cert. |

No error responses other than `500` on allocation failure.

---

## POST /api/system/checkin

Submit a cloud-verified check-in response.

### Request

```
POST /api/system/checkin
Content-Type: application/json
```

```json
{ "checked": "<jwt>" }
```

**Authentication:** Not required — but the JWT inside `checked` must be signed with the same ECDH secret as the GET challenge (i.e. only the publisher's backend can produce it).

### Validation

Performed in order; any failure returns `401`:

1. JWT must decode and HMAC-verify under `Session.session_key` (after the ecdh header causes the key to be re-derived from the `iss` claim).
2. Standard JWT claim checks (`exp`, `iat`) via `jwt_validate()`. If the RTC is unset, `time()` returns `0`, which can invalidate the token — callers relying on check-in to *set* the RTC must ensure `exp` still validates when compared with `0`.
3. `jti` must be a valid time-based nonce issued by **this** device (reject replays and cross-device responses).
4. `iss` must match one of the hardcoded trusted backends (the publisher's two production keys). This check is compiled out in `DEVEL` builds only.

### Claims consumed from `checked`

| Claim | Meaning |
|-------|---------|
| `iat` | Used to set the RTC at the end of the handler (only if later than current RTC offset). First-time RTC set is audit-logged `LOG_TYPE_RTC_SET`. |
| `status` | Base64 HMAC. The device recomputes HMAC-SHA256 over various `(sessdevid || action)` combinations under the session key; the first match selects the action. Matches are: `sessdevid`-only → no action, `+'L'` → **lock**, `+'W'` → **wipeout**, `+'R'` → **report-only**, `+'B'` → **brick**, fallback → **force upgrade** (`U`). |
| `newcrt` | Base64 DER of a replacement TLS certificate for the device's hostname. Processed by `api_post_checkin_newcrt_callback`: on an initialised device, the new cert replaces the old one only if the serial number has **changed**; on a not-yet-initialised device it is saved to `/tmp/tls.crt`. |
| `newfws` | Firmware signature string the broker advertises as available. Echoed into the response for the Manager to drive the upgrade flow. |
| `newuis` | PPA-only. UI bundle signature the broker advertises as available. Echoed into the response. |

### Remote action gating

Lock/wipe/brick actions are **only applied if** [`trusted_backend`](../concepts/device-options.md#trusted_backend--trust-the-cloud-notification-broker) is enabled. When disabled, the device still reports the parsed action in the response but does not execute it.

| Action | Effect on device |
|--------|------------------|
| `L` (lock) | Overwrites `UserKey` with `0x55` bytes (disables user login; Master-Password login still works). Sets `fls_state = 0x40`. Audit-logs `LOG_TYPE_SHUTDOWN`. |
| `W` (wipeout) | Runs `system_full_wipeout()` then reboots into the bootloader in normal mode. |
| `R` (report) | No device-side effect. |
| `B` (brick) | Stops HTTP/TLS web servers. Sets `fls_state = 0x80`. No further auth is possible. Audit-logs `LOG_TYPE_SHUTDOWN`. |
| `U` (force upgrade) | Reserved. No device-side effect in current firmware. |

### Response

**Status:** `200 OK`

```json
{
  "status": "OK",
  "newcrt": "OK",
  "newfws": "<echoed>",
  "newuis": "<echoed>"
}
```

| Field | When emitted | Description |
|-------|--------------|-------------|
| `status` | Always | `"OK"` if no action was encoded; otherwise the single-letter action code (`L`, `W`, `R`, `B`, `U`) or `"???"` on internal error. |
| `newcrt` | Only when request carried `newcrt` | `"OK"` if stored, `"ERROR"` otherwise. |
| `newfws` | Only when request carried `newfws` | Echoed verbatim so the Manager can compare and trigger firmware update. |
| `newuis` | PPA only, and only when request carried `newuis` | Echoed verbatim for UI update flow. |

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON or missing `checked`. |
| 401 | JWT decode/HMAC/jti/iss validation failed. |
| 500 | Internal error. |

## Notes

- The list of trusted backend issuers is **baked into the firmware** (two Ed25519-like Curve25519 public keys in base64). Rotating them requires a firmware release.
- `newcrt` rotation uses cert **serial number** as the change detector — pushing a new cert with the same serial is a silent no-op.
- The whole handler is publicly reachable (no auth). The security model relies on the ECDH signing key (the firmware publisher's private key) being unextractable from the broker backend.

## Encedo Manager Usage

```javascript
// encedo.js:263
self.api('api/system/checkin')
    .then(data => self.api('https://api.encedo.com/checkin', 'POST', data))
    .then(data => self.api('api/system/checkin', 'POST', data))
    .then(function(data) {
        if (data.newfws) self.encedo_update_firmware = data.newfws;
        if (data.newuis) self.encedo_update_dashboard = data.newuis;
    });
```

Manager references: [encedo.js:263](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L263), [build.js:306](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L306), [build.js:1438](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1438)

## Test Suite Usage

```php
// hem-api-tester/test_1.php:87 (GET)
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/checkin", $ret_val, $post_dummy, $token);
// hem-api-tester/test_1.php:148 (POST)
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/system/checkin", $ret_val, $post_data, $token);
```

Test references: [test_1.php:87](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L87), [test_1.php:148](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L148)
