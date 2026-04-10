# GET /api/keymgmt/list/:offset/:limit

List keys stored in the secure repository with pagination.

## Request

```
GET /api/keymgmt/list/:offset/:limit
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `keymgmt:list`

**Allowed users:** User, ExtAuth (Master not allowed)

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `offset` | Number | No | Number of entries to skip (default: 0) |
| `limit` | Number | No | Max entries to return (default: 15) |

### URL Variants

[Encedo Manager](https://github.com/encedo/encedo-manager) uses three URL patterns:
- `api/keymgmt/list` -- no pagination
- `api/keymgmt/list/{offset}` -- offset only
- `api/keymgmt/list/{offset}/{limit}` -- offset + limit

## Response

**Status:** `200 OK`

```json
{
  "offset": 0,
  "total": 25,
  "listed": 15,
  "list": [
    {
      "kid": "09bd0958e1499ecfd51ea62a3f49a84c",
      "created": 1647787070,
      "type": "ED25519",
      "label": "My key",
      "descr": "base64_encoded_description",
      "updated": 1647787070
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `offset` | Number | Starting position |
| `total` | Number | Total number of keys |
| `listed` | Number | Number of keys in this response |
| `list` | Array | Array of key objects |
| `list[].kid` | String | Key ID (32-char hex) |
| `list[].created` | Number | Creation timestamp |
| `list[].type` | String | Key algorithm type |
| `list[].label` | String | Key label |
| `list[].descr` | String | Base64-encoded description |
| `list[].updated` | Number | Last update timestamp |

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
// build.js:6043, core2.js:3748
// Paginated list with offset appended to URL
app.api('api/keymgmt/list' + offset).then(function(data) {
    // data.list contains the key array
    // data.total used for pagination
});
```

**Source references:** [build.js:6043](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6043), [core2.js:3748](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3748)

## Test Suite Usage

```php
// hem-api-tester/test_10.php:68
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/keymgmt/list", $ret_val, "", $token);

// hem-api-tester/test_8.php:300
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/keymgmt/list", $ret_val, "", $token);
```

**Source references:** [test_10.php:68](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L68), [test_8.php:300](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L300)

## Official Docs vs Encedo Manager

**Status:** Documented -- largely matches.

> **DISCREPANCY:** The `_endpoints` map ([build.js:9195](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9195)) registers a **POST** variant of `/api/keymgmt/list` described as "List stored keys (extended version)". The official docs only document the **GET** variant. [Encedo Manager](https://github.com/encedo/encedo-manager) only uses the GET variant with path parameters. The POST variant may accept filter criteria in the body, but this is undocumented.
