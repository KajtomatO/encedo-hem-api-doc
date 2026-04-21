# DELETE /api/keymgmt/delete/{kid}

Permanently remove a key from the on-device repository. Key material is zeroed in flash before the repo slot is released.

## Request

```
DELETE /api/keymgmt/delete/{kid}
Authorization: Bearer <token>
```

**Authentication:** Bearer required, TLS required.

- **Scope:** prefix-match `keymgmt:del`.
- **Subject:** `sub != "M"`.

**Preconditions:** `fls_state==0`, initialised, no software install, valid JWT, TLS.

### Path parameters

| Parameter | Description |
|-----------|-------------|
| `kid` | 32-char hex. |

## Response

**Status:** `200 OK` — no body.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | `kid` missing or not exactly 32 hex chars. |
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope or `sub == "M"`. |
| 406 | `kid` not found in repo. |
| 409 | `fls_state != 0`, software install in progress, or not initialised. |
| 418 | TLS required. |

Audit log: `LOG_TYPE_KEY_DELETE` — `LOG_RESULT_OK` on delete, `LOG_RESULT_ERROR`/`LOG_RESULT_FAILED` on malformed input or missing key.

## Notes

- No confirmation step. Deletion is immediate and irreversible from the API — if the caller deletes a key used by an ExtAuth authenticator, that authenticator stops being able to sign `/ext/request` responses.
- The `kid` is not reused — the repo generates fresh kids via HKDF over the device master secret, so the 16-byte space is effectively unbounded.

## Encedo Manager Usage

```javascript
// build.js:5755
app.api('api/keymgmt/delete/' + kid, 'DELETE');
```

Manager references: [build.js:5755](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5755), [core2.js:3460](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L3460)

## Test Suite Usage

```php
// test_10.php:482
http_transaction("https", "DELETE", $cfg_domain, "/api/keymgmt/delete/" . $kid, $ret_val, "", $token);
```

Test references: [test_10.php:482](https://github.com/encedo/hem-api-tester/blob/main/test_10.php#L482)
