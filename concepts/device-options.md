# Device Options

The device keeps two bitfields of persistent options in its main configuration: `MainCfg.Options` (security/behavioural options) and `MainCfg.http_options` (HTTP stack options). This document describes every field that can be read via `GET /api/system/config` or written via `POST /api/system/config` / `POST /api/auth/init`.

Options are set during initial pairing (`POST /api/auth/init` via the `cfg` JWT payload) and can be updated afterwards via `POST /api/system/config`. See [system/config.md](../system/config.md) and [auth/init.md](../auth/init.md).

---

## Security / behavioural options

### `trusted_ts` — trust the on-device RTC

JSON type: boolean. Default: **true** (PPA and EPA factory defaults).

When **true**, the device trusts its own RTC as the authoritative time source. Time-sensitive endpoints return HTTP `403` whenever the RTC has not been set:
- `GET /api/auth/token`, `POST /api/auth/token`
- `GET /api/auth/init`, `POST /api/auth/init`

Rationale: JWTs carry `iat`/`exp` claims; if the RTC is wrong, expiration checks are meaningless, so the device fails closed rather than issuing or accepting tokens with untrusted timestamps.

When **false**, the device will copy `iat` from an incoming authentication JWT into its own RTC, provided the value is newer than 2021-09 and strictly later than the last stored `iat`. In this mode `GET /api/system/status` advertises the relaxed policy by adding `"tts": 0` to the response.

### `trusted_backend` — trust the cloud notification broker

JSON type: boolean. Default: **true**.

Controls whether the device accepts authentication tokens obtained via `/api/auth/ext/token` (the external/mobile authenticator flow that goes through `api.encedo.com/notify/...`). When **false**, the cloud-broker login path is rejected even if the broker returns a valid signature; only local password authentication (`/api/auth/token`) is usable.

### `allow_keysearch` — enable unauthenticated key search

JSON type: boolean. Default: **true**.

When **true**, `POST /api/keymgmt/search` with a pattern matching the `EXTAID` descriptor prefix is accepted without a Bearer token. This is used by the external-authenticator pairing bootstrap (the mobile app queries the device for its already-paired external identifiers before login). When **false**, the endpoint requires the normal `keymgmt:search` scope.

### `dnsd` — local DNS daemon

JSON type: boolean. Default: **true** on PPA (requires SD-card/USB-MSC build); n/a on EPA.

When **true**, the device answers DNS queries for its own `hostname` on the LAN so clients can reach it by friendly name (e.g. `my.ence.do`) without manual resolver entries. When **false**, clients must reach the device by IP or via external DNS.

### `origin` — CORS origin

JSON type: string (≤63 bytes). Default: `"*"` on PPA factory provisioning.

Value sent in the `Access-Control-Allow-Origin` response header. `"*"` permits all browsers; set to a specific URL (e.g. `https://manager.example.com`) to restrict CORS.

---

## Storage options (PPA only)

### `storage_mode` — storage layout + crypto mode

JSON type: number (bitfield). PPA only.

Composite of two sub-fields (`STORAGE_MODE_*`):

| Bit mask | Values | Meaning |
|----------|--------|---------|
| `0x0F` (disk0) | `0x00` hidden, `0x01` RO, `0x03` RW | Visibility of the plaintext partition (disk0) |
| `0xF0` (disk1 crypto) | `0x50` CTR, `0xA0` XTS | Block-cipher mode for the encrypted partition (disk1) |

Changing the disk1 crypto-mode bits requires a reboot (enforced by firmware in `api_auth.c` and `api_system.c`).

### `storage_disk0size` — plaintext partition size

JSON type: number (sectors). PPA only.

Size of the plaintext (disk0) partition in 512-byte sectors. Clamped by firmware to `sd_card_sector_count - 128KB` so at least 128 KB remains for the encrypted partition. Changing this value requires a reboot.

---

## HTTP stack options

### `http_option_hsts` — HTTP Strict Transport Security header

JSON type: boolean. Default: typically enabled on TLS-reachable devices.

When **true**, TLS responses include `Strict-Transport-Security: max-age=31536000`. Has no effect on plain-HTTP responses.

### `http_option_dosprot_mode` — DoS protection level

JSON type: number, one of `0`, `1`, `2`, `3`, `4`. Default: `0` (off).

Four escalating levels of connection throttling / rate limiting. Higher values apply stricter per-IP and per-endpoint limits. Mode `0` disables the limiter entirely.

---

## Reading / writing options

- `GET /api/system/config` returns the current value of every option documented above (plus identity fields like `user`, `email`, `hostname`, `ip`).
- `POST /api/system/config` accepts a subset of fields; any field omitted from the POST body is left unchanged.
- `POST /api/auth/init` sets the full initial configuration inside the `cfg` sub-object of the auth JWT payload.

All writes are validated (type check + length check via `isvalid_label`) before being committed; a single invalid field causes the whole `POST /api/system/config` to return `400` without partial application.
