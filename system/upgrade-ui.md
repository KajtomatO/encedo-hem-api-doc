# Management App (UI) Upgrade Endpoints

Three-step UI upgrade process: upload, validate, install. Encedo PPA only.

**Scope:** `system:upgrade` (primary), or `system:config` (alternative)

**Allowed users:** User, Master, ExtAuth

---

## POST /api/system/upgrade/upload_ui

Upload a new management app image (webroot.tar).

### Request

```
POST /api/system/upgrade/upload_ui
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/octet-stream
Content-Disposition: attachment; filename="webroot.tar"
Expect: 100-continue
```

**Body:** Raw binary tar file

### Response

**Status:** `200 OK`

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |

---

## GET /api/system/upgrade/check_ui

Validate the uploaded UI image (signature + integrity check).

### Request

```
GET /api/system/upgrade/check_ui
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
| 406 | Operation failed |

---

## GET /api/system/upgrade/install_ui

Install the validated management app.

### Request

```
GET /api/system/upgrade/install_ui
Authorization: Bearer <JWT_TOKEN>
```

### Response

| Code | Description |
|------|-------------|
| 200 | Installation successful |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:1161 -- Upload
xhr2.open('POST', self.encedo_url + '/api/system/upgrade/upload_ui', true);

// build.js:1250 -- Check (polled in setTimeout loop)
xhr.open('GET', self.encedo_url + '/api/system/upgrade/check_ui');

// build.js:1211 -- Install
xhr.open('GET', this.encedo_url + '/api/system/upgrade/install_ui');
```

The UI tarball is downloaded from `api.encedo.com/download/dashboard/{version}`, then uploaded to the device.

**Source references:** [encedo.js:1028](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1028), [build.js:1149](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1149)

## Test Suite Usage

```php
// hem-api-tester/test_7.php:278 (POST upload_ui)
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/system/upgrade/upload_ui", $ret_val, $ui_data, $token);

// hem-api-tester/test_7.php:285 (GET check_ui)
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/check_ui", $ret_val, $post_dummy, $token);

// hem-api-tester/test_7.php:297 (GET install_ui)
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/upgrade/install_ui", $ret_val, $post_dummy, $token);
```

**Source references:** [test_7.php:278](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L278), [test_7.php:285](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L285), [test_7.php:297](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L297)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
