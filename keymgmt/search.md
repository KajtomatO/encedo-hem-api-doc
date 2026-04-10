# POST /api/keymgmt/search

Search the key repository by the `descr` field pattern.

## Request

```
POST /api/keymgmt/search
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `keymgmt:search` (primary), or `keymgmt:list`, `auth:ext:pair` (alternatives)

**Allowed users:** User, ExtAuth (Master not allowed)

**Special:** If `allow_keysearch` is enabled in device config, the `Authorization` header may be omitted when at least 6 bytes of `descr` are provided.

```json
{
  "descr": "search_pattern",
  "offset": 0,
  "limit": 15
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `descr` | String | Yes | Search pattern (applied to key `descr` field) |
| `offset` | Number | No | Skip entries (default: 0) |
| `limit` | Number | No | Max results (default: 15) |

## Response

**Status:** `200 OK`

```json
{
  "offset": 0,
  "total": 5,
  "listed": 5,
  "list": [
    {
      "kid": "09bd0958e1499ecfd51ea62a3f49a84c",
      "created": 1647787070,
      "type": "ED25519",
      "label": "My key",
      "descr": "base64_description",
      "updated": 1647787070
    }
  ]
}
```

Same response format as [/api/keymgmt/list](list.md).

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
// build.js:5806-5837, core2.js:3511-3542
// Search for external authenticator keys by descr pattern
app.api('api/keymgmt/search', 'POST', { descr: '^RVhUQUlE' }).then(function(data) {
    // '^RVhUQUlE' is base64 for '^EXTAID' -- filters for ext auth device keys
});
```

**Source references:** [build.js:5806](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5806), [build.js:5819](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5819), [build.js:5837](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5837), [core2.js:3511](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3511), [core2.js:3524](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3524), [core2.js:3542](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3542)

## Test Suite Usage

```php
// hem-api-tester/test_10.php:410
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/search", $ret_val, $keysearch_arg, $token);

// hem-api-tester/test_11.php:64
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/search", $ret_val, $keysearch_arg, $token);
```

**Source references:** [test_10.php:410](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L410), [test_11.php:64](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L64)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).

> **NOTE:** [Encedo Manager](https://github.com/encedo/encedo-manager) exclusively uses the pattern `'^RVhUQUlE'` (base64 regex for `^EXTAID`) to search for paired external authenticator keys. This reveals that external authenticator keys are stored with a `descr` field prefixed with `EXTAID`.
