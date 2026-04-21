# POST /api/keymgmt/search

Filter the key repository by the `descr` field. Three match modes — substring, prefix (`^`), suffix (`$`) — are selected by the shape of the query.

## Request

```
POST /api/keymgmt/search
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** TLS required. Bearer is **conditionally required** — see below.

### Conditional auth bypass

If **all three** conditions hold, the endpoint is reachable **without** a Bearer:

1. Query is a prefix match (first char of `descr` is `^`).
2. Decoded prefix length ≥ 6 bytes.
3. Device-level [`allow_keysearch`](../concepts/device-options.md#allow_keysearch--enable-unauthenticated-key-search) option is set (i.e. `SYSTEM_OPTION_SECMODE_KEYSEARCH_MASK != SYSTEM_OPTION_SECMODE_KEYSEARCH_FALSE`).

Any other combination requires a valid Bearer whose scope prefix-matches **one** of:

- `keymgmt:search`
- `keymgmt:list`
- `auth:ext:pair`

and whose `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, initialised, TLS.

### Body

```json
{
  "descr":  "<base64_pattern>",
  "offset": 0,
  "limit":  15
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `descr` | String | Yes | Base64-encoded search pattern. Decoded length ≤ `STOREDKEY_DESCR_MAX_LENGTH`. |
| `offset` | Integer | No | Default `0`. |
| `limit` | Integer | No | Default `15`, hard cap `15`. |

### Match modes

The firmware inspects the **first and last characters of the raw `descr` string** before base64 decoding:

| Raw input | Mode | Semantics |
|-----------|------|-----------|
| `^<base64>` | Prefix | Match entries whose stored `descr` **starts with** `base64_decode(rest)`. |
| `<base64>$` | Suffix | Match entries whose stored `descr` **ends with** `base64_decode(rest)`. |
| `<base64>` | Substring | Match entries whose stored `descr` **contains** `base64_decode(input)`. |

Exactly one anchor is allowed. The `^` or `$` character is stripped before base64 decoding.

## Response

**Status:** `200 OK`

Same shape as [`/api/keymgmt/list`](list.md) — `{offset, total, listed, list:[…]}`.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing `descr`, or `descr` base64 decode failure. |
| 401 | Bearer required (bypass conditions not met) and missing. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | Decoded `descr` longer than `STOREDKEY_DESCR_MAX_LENGTH`. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 410 | Repo filter failure. |
| 418 | TLS required. |

## Notes

- The Manager leans on the **unauthenticated bypass path** to discover paired authenticators before the user has logged in: it issues a search for `^EXTAID` (base64 `^RVhUQUlE`), gets the list of paired-app kids, and uses those to drive the login UI.
- The bypass is gated on `allow_keysearch` being enabled at init time ([`/api/auth/init`](../auth/init.md)) and can be retoggled later via `POST /api/system/config`. A device that left `allow_keysearch` off will always require a Bearer, even for `^EXTAID` lookups.

## Encedo Manager Usage

```javascript
// build.js:5806
app.api('api/keymgmt/search', 'POST', { descr: '^RVhUQUlE' });  // '^EXTAID'
```

Manager references: [build.js:5806](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5806), [core2.js:3511](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3511)

## Test Suite Usage

```php
// test_10.php:410, test_11.php:64
http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/search", $ret_val, $keysearch_arg, $token);
```

Test references: [test_10.php:410](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L410), [test_11.php:64](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L64)
