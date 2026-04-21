# GET /api/system/selftest

Run the device self-test battery (KAT — Known-Answer Tests, entropy check, FLS state) and return the key-repository statistics. Calling this endpoint is the main trigger for `fls_state` to update.

## Request

```
GET /api/system/selftest
```

**Authentication:**
- On an **initialised** device: Bearer token required (any valid token — no scope check).
- On a **fresh/uninitialised** device: no auth.

## Response

**Status:** `200 OK`

```json
{
  "last_selftest_ts": 1705312200,
  "last_fls_state": 0,
  "last_entropytest_ts": 1705311000,
  "last_kat_ts": 1705300000,
  "kat_busy": true,
  "fls_state": 0,
  "selftest_ts": 1705312300,
  "repo_stats": {
    "total": 128,
    "deleted": 5,
    "invalid": 0,
    "fragmented": 2,
    "freeslots": 867
  },
  "se_state": 0
}
```

| Field | Type | When emitted | Description |
|-------|------|--------------|-------------|
| `last_selftest_ts` | Number | Always | Timestamp of the previous self-test run (before *this* one). |
| `last_fls_state` | Number | Always | Fail-state code at the previous run. |
| `last_entropytest_ts` | Number | Always | Timestamp of the most recent entropy self-test. |
| `last_kat_ts` | Number | Always | Timestamp of the most recent KAT pass. |
| `kat_busy` | Boolean `true` | Only when `SELFTEST_KAT_Status()` reports in-progress | Present only if KAT is running; absent otherwise. |
| `fls_state` | Number | Always | Current fail-state after the re-run triggered by this call. `0` = nominal. |
| `selftest_ts` | Number | Always | Timestamp of **this** run. |
| `repo_stats.total` | Number | Always | Total keys in the on-device repository. |
| `repo_stats.deleted` | Number | Always | Logically deleted slots (reclaimed on compaction). |
| `repo_stats.invalid` | Number | Always | Slots flagged invalid by integrity check. |
| `repo_stats.fragmented` | Number | Always | Slots needing compaction. |
| `repo_stats.freeslots` | Number | Always | Free slots remaining. |
| `se_state` | Number | PPA only | ATECC608 secure-enclave status. |

### Side effects

- Always runs `self_check(0xFF, now)` — re-executes the full self-test battery.
- Updates `Session.selftest_user_ts` to `now`.
- If the RTC was set after previous runs, past timestamps are retro-offset so the returned values are absolute epoch timestamps rather than pre-RTC monotonic counts.
- Writes an audit-log event `LOG_TYPE_SELFTEST_PASSED` with result `OK` or `FAILED` based on `fls_state`.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device is initialised and Bearer missing/invalid. |
| 500 | JSON allocation failure. |

## Notes

- `repo_stats` is **only** returned by this endpoint — it is **not** included in `GET /api/system/status`.
- Calling this endpoint is the canonical way to force a self-test re-run; there is no separate "run" endpoint.

## Encedo Manager Usage

```javascript
app.api('api/system/selftest').then(function(data) {
    self.encedo_fls_state = data.fls_state;
    self.encedo_repo_stats = data.repo_stats;
});
```

Manager references: [encedo.js:313](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L313), [build.js:356](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L356)

## Test Suite Usage

```php
// test_0.php:49
http_transaction("https", "GET", $cfg_domain, "/api/system/selftest", $ret_val, $post_dummy, $token);
```

Test references: [test_0.php:49](https://github.com/encedo/hem-api-tester/blob/main/test_0.php#L49), [test_1.php:51](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L51)
