# Manager UI Upgrade Endpoints

PPA-only, three-step UI bundle upgrade: **upload → check → install**. The "UI" is the Manager web-app tarball served from `/webroot` on the device.

- **Upload scope:** `system:upgrade`.
- **Check/install scopes:** `system:upgrade` or `system:config`.
- On an **uninitialised** device the flow is available without a Bearer (first-boot provisioning).
- Another install in progress (firmware or UI) → `409`.

The UI bundle is a tar file (`webroot.tar`) containing a `manifest` whose SHA-256 hash is recorded as `Session.ui_ver_hash` on successful install. That hash is what `GET /api/system/version` returns in `uis` and what `GET /api/system/checkin` sends to the broker.

---

## POST /api/system/upgrade/upload_ui

Upload `webroot.tar` to `/tmp/`.

### Request

```
POST /api/system/upgrade/upload_ui
Authorization: Bearer <token>
Content-Disposition: attachment; filename="webroot.tar"
Content-Type: application/octet-stream
Expect: 100-continue
```

- Absolute cap on any file-upload POST is 16 MiB → `413`.
- No extension/content check beyond the generic 16 MiB limit; the signature check happens in the subsequent `check_ui` step.

### Response

`200 OK` on successful upload.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope not `system:upgrade`. |
| 413 | >16 MiB. |

---

## GET /api/system/upgrade/check_ui

Asynchronously validate the tarball. Verifies an Ed25519 signature over the file (`ed25519_verify_file`), unpacks it into `/webroot.new/`, then runs `process_manifest_file` to hash-check every file listed in the manifest.

### Request

```
GET /api/system/upgrade/check_ui
Authorization: Bearer <token>
```

### Response (empty body; status is the signal)

| `check_ui_status` | Returned status |
|-------------------|-----------------|
| 0 (idle) — this call starts the task | `201` |
| 1 (task running) | `202` |
| 2 (starting) | `202` |
| 3 (finished) — `/webroot.new/` exists | `200` |
| 3 (finished) — `/webroot.new/` missing | `406` |

After a finished read (`200`/`406`), the status resets to `0` — the next call will start a new check.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope mismatch. |
| 409 | Another state prevented the call. |

---

## GET /api/system/upgrade/install_ui

Promote `/webroot.new/` to `/webroot/`, archive the previous `/webroot/` as `/webroot.old/`, and recompute `Session.ui_ver_hash` from the new manifest.

### Request

```
GET /api/system/upgrade/install_ui
Authorization: Bearer <token>
```

**Preconditions:**
- No other install in progress (otherwise `409`).
- `/webroot.new/` must exist (produced by a passing `check_ui`). Missing → `406`.

### Response

`200 OK` on success. Audit-logged `LOG_TYPE_UPGRADE` (OK or FAILED). Unlike firmware install, this is **not** followed by a reboot — the new UI is immediately served.

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Device initialised and no Bearer. |
| 403 | Scope mismatch. |
| 406 | `/webroot.new/` missing (check_ui never passed). |
| 409 | Another install in progress. |

## Encedo Manager Usage

```javascript
// build.js:1161 — upload
xhr.open('POST', self.encedo_url + '/api/system/upgrade/upload_ui', true);
// build.js:1250 — poll check_ui
xhr.open('GET', self.encedo_url + '/api/system/upgrade/check_ui');
// build.js:1211 — install_ui
xhr.open('GET', self.encedo_url + '/api/system/upgrade/install_ui');
```

Manager references: [encedo.js:1028](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L1028), [build.js:1149](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1149)

## Test Suite Usage

```php
// test_7.php:278 (upload) / :285 (check) / :297 (install)
http_transaction("https", "POST", $cfg_domain, "/api/system/upgrade/upload_ui", $ret_val, $ui_data, $token);
http_transaction("https", "GET",  $cfg_domain, "/api/system/upgrade/check_ui",  $ret_val, $post_dummy, $token);
http_transaction("https", "GET",  $cfg_domain, "/api/system/upgrade/install_ui", $ret_val, $post_dummy, $token);
```

Test references: [test_7.php:278](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L278), [test_7.php:285](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L285), [test_7.php:297](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L297)
