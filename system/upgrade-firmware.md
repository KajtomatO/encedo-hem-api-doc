# Firmware Upgrade Endpoints

Three-step firmware upgrade process: upload, validate, install.

**Scope:** `system:upgrade` (primary), or `system:config` (alternative)

**Allowed users:** User, Master, ExtAuth

---

## POST /api/system/upgrade/upload_fw

Upload a new firmware image file.

### Request

```
POST /api/system/upgrade/upload_fw
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/octet-stream
Content-Disposition: attachment; filename="firmware.bin"
Expect: 100-continue
```

**Body:** Raw binary firmware file

### Response

**Status:** `200 OK`

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |

---

## GET /api/system/upgrade/check_fw

Validate the uploaded firmware file (integrity + signature check).

### Request

```
GET /api/system/upgrade/check_fw
Authorization: Bearer <JWT_TOKEN>
```

### Response

| Code | Description |
|------|-------------|
| 200 | Validation successful |
| 201 | Validation process started |
| 202 | Validation ongoing (poll again) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed (invalid firmware) |

---

## GET /api/system/upgrade/install_fw

Install the validated firmware. Device reboots on success.

### Request

```
GET /api/system/upgrade/install_fw
Authorization: Bearer <JWT_TOKEN>
```

### Response

| Code | Description |
|------|-------------|
| 200 | Installation successful -- device reboots |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:976-1020 -- Upload (with retry logic)
xhr2.open('POST', self.encedo_url + '/api/system/upgrade/upload_fw', true);

// build.js:1111 -- Check
xhr.open('GET', this.encedo_url + '/api/system/upgrade/check_fw');

// build.js:1075 -- Install
xhr.open('GET', this.encedo_url + '/api/system/upgrade/install_fw');
```

The firmware binary is first downloaded from `api.encedo.com/download/firmware/{version}/bin`, then uploaded to the device.

**Source references:** [encedo.js:855](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L855), [build.js:976](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L976)

## Test Suite Usage

```php
// hem-api-tester/test_7.php:184 (POST upload_fw)
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/system/upgrade/upload_fw", $ret_val, $fw_data, $token);

// hem-api-tester/test_7.php:191 (GET check_fw)
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/check_fw", $ret_val, $post_dummy, $token);

// hem-api-tester/test_7.php:203 (GET install_fw)
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/install_fw", $ret_val, $post_dummy, $token);
```

**Source references:** [test_7.php:184](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L184), [test_7.php:191](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L191), [test_7.php:203](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L203)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **NOTE:** [Encedo Manager](https://github.com/encedo/encedo-manager) includes retry logic for upload -- if a chunk upload fails, it re-attempts from the beginning. The `check_fw` endpoint is polled in a loop waiting for status 200 (validation complete).
