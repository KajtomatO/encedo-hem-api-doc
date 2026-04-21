# GET /api/keymgmt/list[/{offset}[/{limit}]]

Paginated listing of every key in the repository. Returns metadata only; use [`/api/keymgmt/get/{kid}`](get.md) to retrieve public material.

## Request

```
GET /api/keymgmt/list                   # offset=0, limit=15
GET /api/keymgmt/list/20                # skip 20, limit=15
GET /api/keymgmt/list/20/10             # skip 20, limit=10 (capped to 15)
Authorization: Bearer <token>
```

**Authentication:** Bearer required, TLS required.

- **Scope:** prefix-match `keymgmt:list`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, valid JWT, TLS.

### Path parameters

| Parameter | Type | Default | Notes |
|-----------|------|---------|-------|
| `offset` | Integer | `0` | Skip the first N entries. |
| `limit` | Integer | `15` | Hard-capped server-side at `15`. Asking for more returns 15. |

Malformed / non-integer suffix segments return `406`.

## Response

**Status:** `200 OK`

```json
{
  "offset": 20,
  "total":  100,
  "listed": 10,
  "list": [
    {
      "kid":     "09bd0958e1499ecfd51ea62a3f49a84c",
      "created": 1647787070,
      "updated": 1647787070,
      "type":    "ED25519",
      "label":   "My signing key",
      "descr":   "<base64_blob>"
    }
  ]
}
```

| Field | Description |
|-------|-------------|
| `offset` | Echo of the effective offset. |
| `total` | Total number of keys in the repo. |
| `listed` | Number of entries in `list` (≤ `limit`). |
| `list[].kid` | 32-char hex key id. |
| `list[].type` | Algorithm string (same vocabulary as [get](get.md)). |
| `list[].label` | Human label as stored. |
| `list[].descr` | Base64 opaque blob; absent if empty. |
| `list[].created` / `list[].updated` | Unix timestamps. |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | Non-integer path suffix, or repo list failure. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

## Notes

- The **15-item cap** is a server limit; to walk the whole repo, paginate by passing increasing `offset`.
- Listing returns *every* key in the repo, including the user-supplied Curve25519 persona keys planted at init (`masterkey`, `userkey`), ExtAuth paired-authenticator public keys (descriptor-prefix `EXTAID`), and TLS certs. Filter client-side by `type`, `label`, or the `descr` field, or use [`/api/keymgmt/search`](search.md) for server-side `descr` filtering.

## Encedo Manager Usage

```javascript
// build.js:6043
app.api('api/keymgmt/list' + offset).then(data => { /* data.list, data.total */ });
```

Manager references: [build.js:6043](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6043), [core2.js:3748](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3748)

## Test Suite Usage

```php
// test_10.php:68
http_transaction("https", "GET", $cfg_domain, "/api/keymgmt/list", $ret_val, "", $token);
```

Test references: [test_10.php:68](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L68), [test_8.php:300](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L300)
