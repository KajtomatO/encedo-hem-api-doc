# GET /api/system/status

Retrieve device operational status, uptime, temperature, and storage state.

## Request

```
GET /api/system/status
```

**Authentication:** Not required. No scope check. A Bearer token, if provided, is ignored here — the response contents do not change based on caller identity.

## Response

**Status:** `200 OK`

Fields are emitted **conditionally** — the exact JSON shape depends on device hardware (PPA vs EPA), TLS availability, and runtime state. The response never contains all of the fields below simultaneously.

```json
{
  "inited": false,
  "https": false,
  "hostname": "my.ence.do",
  "tts": 0,
  "ctx": 0,
  "uptime": 3600,
  "ts": "2024-01-15T10:30:00Z",
  "time": 1705312200,
  "storage": ["8388607:rw", "8388607:-"],
  "format": "status3",
  "temp": 35,
  "fls_state": 0,
  "fw_upgrade": true
}
```

| Field | Type | When emitted | Description |
|-------|------|--------------|-------------|
| `inited` | Boolean | Only when device is **not** initialised | Always `false` when present; when the device is initialised the key is omitted entirely. Callers should treat "key absent" as `inited: true`. |
| `https` | Boolean | Only when the request arrived over **plain HTTP** | `true` = HTTPS listener is up and the caller may upgrade; `false` = HTTPS not yet available (e.g. no provisioned TLS cert). The field is deliberately absent on TLS-reachable requests. |
| `hostname` | String | Only when the HTTP `Host` header does **not** start with the configured device hostname | Lets the Manager detect misrouted requests and redirect to the configured hostname. |
| `tts` | Number `0` | Only when [`trusted_ts`](../concepts/device-options.md#trusted_ts--trust-the-on-device-rtc) option is **false** | Explicit marker that the device is running in untrusted-time mode (i.e. accepts client-supplied `iat` to prime its RTC). When `trusted_ts` is true the field is omitted. |
| `ctx` | Number | Always | Instance context id (`MainCfg.context`), set on init and updatable via `POST /api/system/config`. Opaque to the device — provided for caller-side correlation. |
| `uptime` | Number | Always | Seconds since boot. Primary "device reachable" signal for polling clients. |
| `ts` | String | Only when RTC is set | Current device time in `YYYY-MM-DDTHH:MM:SSZ` format. |
| `time` | Number | Only when RTC is set | Current Unix timestamp (seconds). |
| `storage` | Array of string | PPA only (`USB_MSC_AVAILABLE` build) | Two entries, one per onboard microSD slot, in the form `"<sector_count-1>:<mode>"` where mode is `rw`, `ro`, or `-` (slot not ready). |
| `format` | String | PPA only, while a format operation is in progress | `"status1"`…`"status9"` mirror the running formatter state; `"done"` once complete. Field is absent when idle. |
| `temp` | Number | Always | Die temperature from `Session.temperature` (°C). |
| `fls_state` | Number | Always | Fail-state code from `Session.fls_status`. `0` = nominal. Non-zero values indicate a self-test or integrity failure; high bits (`0x40`, `0x80`) are set by the remote-lock/brick flow in `POST /api/system/checkin`. |
| `fw_upgrade` | Boolean `true` | Only on the first status read after a successful firmware upgrade | Indicates the device has just rebooted into a newly installed firmware. Consumed/cleared by the boot sequence. |

### Notes on caller behaviour

- The response is intentionally **sparse** — do not require any field except `uptime`, `temp`, `fls_state`, and `ctx` in parsers. Specifically, `inited`, `https`, and `tts` only appear when they carry information.
- Key-repository statistics (`repo_stats`) are **not** returned by this endpoint on current firmware; they are served by `GET /api/system/selftest` only. See [system/selftest.md](selftest.md).

## Encedo Manager Usage

```javascript
// Connection detection, status polling, storage UI
app.api('api/system/status').then(function(data) {
    // data.uptime     → "device reachable"
    // data.inited     → "absent" means initialised; if false, show the init wizard
    // data.https      → if present and true, redirect HTTP→HTTPS
    // data.storage[0] → "<sectors>:rw|ro|-" for disk0 UI
});
```

Manager references: [encedo.js:222](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L222), [build.js:265](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L265), [build.js:3510](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L3510), [build.js:7710](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7710), [core2.js:1257](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L1257)

## Test Suite Usage

```php
// hem-api-tester/test_0.php:37
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/status", $ret_val, $post_dummy, $token);
```

Test references: [test_0.php:37](https://github.com/encedo/hem-api-tester/blob/main/test_0.php#L37), [test_1.php:39](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L39), [test_7.php:56](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L56), [test_8.php:34](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L34)
