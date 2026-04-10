# GET /api/system/status

Retrieve device operational status, uptime, temperature, and storage state.

## Request

```
GET /api/system/status
```

**Authentication:** Not required (optional -- required for key repository statistics)

**Headers:** None required

## Response

**Status:** `200 OK`

```json
{
  "ctx": 1,
  "fls_state": 0,
  "uptime": 3600,
  "ts": "2024-01-15T10:30:00Z",
  "time": 1705312200,
  "temp": 35,
  "storage": ["disk0_status", "disk1_status"],
  "format": "formatting_state",
  "fw_upgrade": true,
  "inited": true,
  "https": true,
  "hostname": "my.ence.do",
  "repo_stats": {
    "deleted": 0,
    "fragmentation": 0,
    "freespace": 1024,
    "invalid": 0,
    "total": 100
  },
  "tts": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ctx` | Number | Context number |
| `fls_state` | Number | Fail state value (0 = no errors) |
| `uptime` | Number | Seconds since device boot |
| `ts` | String | Current time in ISO 8601 (if RTC is set) |
| `time` | Number | Current Unix timestamp (if RTC is set) |
| `temp` | Number | Chip temperature in Celsius |
| `storage` | Array | Capacity and status of each embedded disk |
| `format` | String | Optional. Disk formatting state during personalization |
| `fw_upgrade` | Boolean | Optional. Indicates successful reboot post-upgrade |
| `inited` | Boolean | Optional. `false` if device is unpersonalized |
| `https` | Boolean | Optional. HTTPS availability |
| `hostname` | String | Optional. Device hostname |
| `repo_stats` | Object | Key repository memory stats (requires auth) |
| `tts` | Boolean | Optional. `false` if TrustedTime is disabled |

## Encedo Manager Usage

```javascript
// Used for connection detection, status polling, storage state checking
// encedo.js:222, build.js:265, build.js:3510, build.js:7710-7718
app.api('api/system/status').then(function(data) {
    // data.uptime used to detect device presence
    // data.storage used to show disk states
    // data.inited used to determine init vs auth flow
});
```

**Source references:** [encedo.js:222](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L222), [build.js:265](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L265), [build.js:347](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L347), [build.js:3510](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L3510), [build.js:7710](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7710), [core2.js:1257](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L1257), [core2.js:5418](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5418), launch.html:91

## Test Suite Usage

```php
// hem-api-tester/test_0.php:37
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/status", $ret_val, $post_dummy, $token);
```

**Source references:** [test_0.php:37](https://github.com/encedo/hem-api-tester/blob/main/test_0.php#L37), [test_1.php:39](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L39), [test_7.php:56](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L56), [test_8.php:34](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L34)

## Notes

- This endpoint is polled continuously for connection status monitoring.
- The `uptime` field is the primary indicator that the device is connected and responsive.
- The `inited` field determines whether to show the personalization wizard or the login screen.
- The `storage` array is used to display disk0/disk1 lock/unlock states.

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
