# POST /api/keymgmt/update

Update a key's label and/or description fields.

## Request

```
POST /api/keymgmt/update
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:upd`

**Allowed users:** User, ExtAuth (Master not allowed)

```json
{
  "kid": "32char_hex_key_id",
  "label": "New label",
  "descr": "base64_new_description"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `kid` | String | Yes | Key ID (32-char hex) |
| `label` | String | No | New label |
| `descr` | String | No | Base64-encoded new description |

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
// build.js:5901, build.js:6604, build.js:6706
// Update key label
app.api('api/keymgmt/update', 'POST', {
    kid: openedKey.kid,
    label: key_label_new.value,
    descr: openedKey.descr
});

// Update after creation (add description post-create)
// build.js:6706
app.api('api/keymgmt/update', 'POST', newKeyData);
```

**Source references:** [build.js:5901](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5901), [build.js:6604](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6604), [build.js:6706](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6706), [core2.js:3606](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3606), [core2.js:4309](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4309), [core2.js:4411](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4411)

## Test Suite Usage

```php
// hem-api-tester/test_10.php:362
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/update", $ret_val, $keyupdate_arg, $token);
```

**Source references:** [test_10.php:362](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L362)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
