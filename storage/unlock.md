# GET /api/storage/unlock[/{ro|rw}]

Mount an embedded microSD partition for host-side access over USB-MSC. **PPA only** (compiled behind `USB_MSC_AVAILABLE`; EPA returns `404`).

The disk number (`0` or `1`) is determined from the JWT **scope suffix**, not the URL path. The optional URL suffix (`/ro` or `/rw`) lets the caller request a narrower access mode than the scope allows.

## Request

```
GET /api/storage/unlock            # mode = from scope (rw if ":rw" present)
GET /api/storage/unlock/ro         # force read-only (requires scope with or without ":rw")
GET /api/storage/unlock/rw         # force read-write (requires scope ending with ":rw")
Authorization: Bearer <token>
```

### Scopes

The scope prefix is literal `storage:disk` followed by a digit and optional `:rw`:

| Scope | Disk | Mode |
|-------|------|------|
| `storage:disk0` | Plaintext partition | Read-only |
| `storage:disk0:rw` | Plaintext partition | Read-write |
| `storage:disk1` | Encrypted partition (`StoragePlainSize < SDCardSize`, see [`storage_mode`](../concepts/device-options.md#storage_mode--storage-layout--crypto-mode)) | Read-only |
| `storage:disk1:rw` | Encrypted partition | Read-write |

Subject (`sub`) must **not** be `"M"` — master-key tokens cannot unlock disks.

**Preconditions:**
1. `fls_state == 0` — otherwise `409`.
2. Initialised — otherwise `409`.
3. No disk format in progress — otherwise `409`.
4. Valid Bearer — otherwise `401`.

### Path/scope combinations

| URL suffix | Scope form | Resulting mode |
|------------|------------|----------------|
| *(none)* | `storage:diskN` | Read-only |
| *(none)* | `storage:diskN:rw` | Read-write |
| `/ro` | either | Read-only |
| `/rw` | `storage:diskN:rw` | Read-write |
| `/rw` | `storage:diskN` (no `:rw`) | `406` |
| anything else | — | `406` |

In `DEBUG` builds, disk `2` is also accepted (third card slot). Production builds accept only `0` and `1`.

## Response

**Status:** `200 OK` on success. No body.

The device sets `sd_card_map[disk_no].writable` (0/1) and `sd_card_map[disk_no].ready = 2`, which causes the USB-MSC task to expose the partition to the connected host.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing/invalid Bearer. |
| 403 | Scope doesn't start with `storage:disk` or `sub == "M"`. |
| 406 | Scope digit not `0`/`1`, path suffix not `/ro`/`/rw`, or `/rw` asked with a non-`:rw` scope. |
| 409 | `fls_state != 0`, not initialised, or disk format in progress. |

No success audit event is emitted — only scope failures are logged (`LOG_TYPE_FAILED_SCOPE_CHECK`).

## Notes

- **"Unlock" is not a crypto operation here.** For disk 1 (encrypted), the key material is already in RAM once the device is initialised and selftest passes; the endpoint only flips the USB-MSC visibility flags. Decryption-at-rest happens transparently in the hardware XTS engine during USB reads/writes.
- The plain `GET /api/storage/unlock` path with no suffix is supported and matches what the test suite uses. The `/ro` and `/rw` path variants exist as an override mechanism for the Manager UI.

## Encedo Manager Usage

```javascript
// build.js:7729-7817
var endpoint = writePermission ? 'api/storage/unlock/rw' : 'api/storage/unlock/ro';
app.api(endpoint).then(() => app.api('api/system/status'));
```

Manager references: [build.js:7729](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L7729), [core2.js:5437](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L5437)

## Test Suite Usage

```php
// test_12.php:77
http_transaction("https", "GET", $cfg_domain, "/api/storage/unlock", $ret_val, "", $token);
```

Test references: [test_12.php:77](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L77), [test_12.php:87](https://github.com/encedo/hem-api-tester/blob/main/test_12.php#L87)
