# GET /api/system/selftest

Execute an internal self-test of the device's critical components, including cryptography primitives, random number generator, and key repository integrity.

## Request

```
GET /api/system/selftest
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** Any valid access token

**Allowed users:** User, Master, ExtAuth

## Response

**Status:** `200 OK`

```json
{
  "last_selftest_ts": 1647615622,
  "last_fls_state": 0,
  "last_entropytest_ts": 1647615622,
  "last_kat_ts": 1647606250,
  "kat_busy": true,
  "fls_state": 0,
  "selftest_ts": 1647615642,
  "se_state": 0
}
```

| Field | Type | Description |
|-------|------|-------------|
| `last_selftest_ts` | Number | Timestamp of last self-test |
| `last_fls_state` | Number | Previous fail state |
| `last_entropytest_ts` | Number | Timestamp of last entropy test |
| `last_kat_ts` | Number | Timestamp of last Known Answer Test |
| `kat_busy` | Boolean | Whether KAT is currently running |
| `fls_state` | Number | Current fail state (0 = OK) |
| `selftest_ts` | Number | Current self-test timestamp |
| `se_state` | Number | Secure Enclave state |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:5069, core2.js:2807
app.api('api/system/selftest').then(function(data) {
    // Used in hardware status page to display test results
});
```

**Source references:** [build.js:5069](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5069), [core2.js:2807](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2807)

## Test Suite Usage

```php
// hem-api-tester/test_8.php:67
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/selftest", $ret_val, $post_dummy, $token);
```

**Source references:** [test_8.php:67](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L67), [test_8.php:76](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L76), [test_8.php:336](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L336), [test_8.php:345](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L345)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **DISCREPANCY (minor):** The official docs list the response fields including `repo_stats` sub-object (total, deleted, invalid, fragmented, freeslots), but [Encedo Manager](https://github.com/encedo/encedo-manager) doesn't seem to specifically reference those nested fields from this endpoint.
