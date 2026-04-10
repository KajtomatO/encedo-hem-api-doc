# DELETE /api/keymgmt/delete/:kid

Delete a key from the secure repository.

## Request

```
DELETE /api/keymgmt/delete/:kid
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `keymgmt:del`

**Allowed users:** User, ExtAuth (Master not allowed)

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `kid` | String | Key ID, 32-character hex string |

## Response

**Status:** `200 OK`

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Incorrect argument(s) |
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 406 | Operation failed |
| 409 | Incorrect internal state |
| 418 | TLS connection required |

## Encedo Manager Usage

```javascript
// build.js:1881, build.js:5755, build.js:5970-5977, build.js:6259
app.api('api/keymgmt/delete/' + kid, 'DELETE');
```

**Source references:** [build.js:1881](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1881), [build.js:5755](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5755), [build.js:5970](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5970), [build.js:6259](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6259), [core2.js:3460](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3460), [core2.js:3675](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3675), [core2.js:3964](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3964)

## Test Suite Usage

```php
// hem-api-tester/test_10.php:482
$ret_stat = http_transaction("https", "DELETE", $cfg_domain, "/api/keymgmt/delete/" . $kid, $ret_val, "", $token);
```

**Source references:** [test_10.php:482](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L482)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
