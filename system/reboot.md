# GET /api/system/reboot

Reboot the device. Invalidates all issued access tokens.

## Request

```
GET /api/system/reboot
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `system:upgrade` (primary), or `system:config`, `system:shutdown` (alternatives)

**Allowed users:** User, Master, ExtAuth

## Response

**Status:** `200 OK` -- Device reboots

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// build.js:1911
return self.api('api/system/reboot');
```

**Source references:** [build.js:1911](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1911)

## Test Suite Usage

```php
// hem-api-tester/test_2.php:195
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/reboot", $ret_val, $post_dummy, $token);
```

**Source references:** [test_2.php:195](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L195), [test_8.php:149](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L149), [test_9.php:88](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L88)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
