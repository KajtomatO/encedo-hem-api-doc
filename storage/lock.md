# GET /api/storage/lock

Withdraw one of the embedded partitions from host-side USB-MSC access. **PPA only** (compiled behind `USB_MSC_AVAILABLE`; EPA returns `404`).

The disk number is encoded in the JWT **scope suffix** (digit immediately after `storage:disk`). There is no URL argument.

## Request

```
GET /api/storage/lock
Authorization: Bearer <token>
```

### Scopes

Scope must start with `storage:disk` followed by a digit:

- `storage:disk0` → lock disk 0.
- `storage:disk1` → lock disk 1.
- (DEBUG builds) `storage:disk2` → lock disk 2.

Subject (`sub`) must **not** be `"M"`.

**Preconditions:**
1. `fls_state == 0` — otherwise `409`.
2. Initialised — otherwise `409`.
3. No disk format in progress — otherwise `409`.
4. Valid Bearer — otherwise `401`.

## Response

**Status:** `200 OK` on success. No body.

The device sets `sd_card_map[disk_no].writable = 0` and `sd_card_map[disk_no].ready = 0`, causing the USB-MSC task to withdraw the partition.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing/invalid Bearer. |
| 403 | Scope doesn't start with `storage:disk` or `sub == "M"`. |
| 406 | Scope digit not `0`/`1` (or `2` in DEBUG). |
| 409 | `fls_state != 0`, not initialised, or disk format in progress. |

## Notes

- Locking does not re-encrypt or zero any content — it just flips the USB-MSC visibility flags. For the encrypted partition (disk 1) the persistent state is always encrypted; locking only hides it from the host.
- "Lock all disks at once" is not a supported single call — issue one `GET /api/storage/lock` per disk (using a token whose scope references the target disk).

## Encedo Manager Usage

```javascript
// build.js:7848
app.api('api/storage/lock').then(() => app.api('api/system/status'));
```

Manager references: [build.js:7848](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7848), [core2.js:5556](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5556)

## Test Suite Usage

```php
// test_12.php:69
http_transaction("https", "GET", $cfg_domain, "/api/storage/lock", $ret_val, "", $token);
```

Test references: [test_12.php:69](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L69), [test_12.php:118](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L118)
