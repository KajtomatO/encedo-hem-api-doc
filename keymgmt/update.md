# POST /api/keymgmt/update

Rewrite the `label` and/or `descr` metadata of an existing key. Does not rotate key material.

## Request

```
POST /api/keymgmt/update
Authorization: Bearer <token>
Content-Type: application/json
```

**Authentication:** Bearer required, TLS required.

- **Scope:** prefix-match `keymgmt:upd`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, no software install, valid JWT, TLS, initialised.

### Body

```json
{
  "kid":   "<32_char_hex_key_id>",
  "label": "<new_label>",
  "descr": "<base64_opaque_blob>"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| `kid` | String | Yes | 32-char hex. |
| `label` | String | Yes | `isvalid_label` ≤32 bytes. |
| `descr` | String | No | Base64 blob, ≤`STOREDKEY_DESCR_MAX_LENGTH` bytes decoded. Omit to leave existing descr unchanged. |

Despite the firmware's fallback check for "at least one of label/descr", `label` is required at parse time — leaving it out returns `400`.

## Response

**Status:** `200 OK` — no body.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, invalid `kid`/`label`, or `descr` base64 decode failure. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | `kid` not found in repo. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_KEY_ATTRIBUTES_CHANGED`.

## Notes

- There is no partial update for `label` alone if you also want to clear `descr` — the firmware updates whichever fields you send, and leaves the others untouched when omitted.

## Encedo Manager Usage

```javascript
// build.js:5901
app.api('api/keymgmt/update', 'POST', { kid, label, descr });
```

Manager references: [build.js:5901](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5901), [build.js:6604](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6604), [core2.js:3606](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3606)

## Test Suite Usage

```php
// test_10.php:362
http_transaction("https", "POST", $cfg_domain, "/api/keymgmt/update", $ret_val, $keyupdate_arg, $token);
```

Test references: [test_10.php:362](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L362)
