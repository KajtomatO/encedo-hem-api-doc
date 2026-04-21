# GET /api/system/version

Retrieve device hardware, firmware, and bootloader identity/signatures.

## Request

```
GET /api/system/version
```

**Authentication:** Not required. A valid Bearer token unlocks two additional fields on PPA hardware (`sd_csd`, `sd_cid`) but is otherwise optional.

## Response

**Status:** `200 OK`

```json
{
  "hwv": "PPA rev 2.2",
  "fwv": "Encedo nGINE FW v1.2.2",
  "fwk": "base64_firmware_public_key",
  "fws": "base64_firmware_signature",
  "blv": "Encedo Secure Bootloader v2.0.1",
  "blk": "base64_bootloader_public_key",
  "bls": "base64_bootloader_signature",
  "sd_csd": "hex_sd_csd",
  "sd_cid": "hex_sd_cid",
  "uis": "base64_ui_hash"
}
```

| Field | Type | When emitted | Description |
|-------|------|--------------|-------------|
| `hwv` | String | Always | Hardware identity string. `"PPA rev 2.2"`, `"EPA rev 1.0 @<mcu_id>"`, or `"devboard"`. The Manager uses substring `"EPA"` to distinguish device families. |
| `fwv` | String | Always | Firmware name read from `MyFooter.MyName` (signed footer appended to the firmware image). |
| `fwk` | String | Always | Application signing public key (base64) from `MyFooter.ApplicationPublicKey`. |
| `fws` | String | Always | Signature over the firmware image (base64) from `MyFooter.Signature`. |
| `blv` | String | Only if a bootloader footer is present and signed by the same publisher | Bootloader name. Absent when the bootloader footer isn't found or doesn't match the firmware publisher. |
| `blk` | String | Same condition as `blv` | Bootloader signing public key (base64). |
| `bls` | String | Same condition as `blv` | Bootloader signature (base64). |
| `sd_csd` | String | PPA only, and only when device is **not initialised** or a valid JWT is presented | microSD CSD register dumped as a contiguous hex string (32 chars per 16-byte CSD). |
| `sd_cid` | String | Same condition as `sd_csd` | microSD CID register as hex. |
| `uis` | String | PPA only | Base64 of the hash of the currently installed Manager UI (`Session.ui_ver_hash`). Used by the Manager to detect when to fetch a newer bundle. |

No error responses other than `500` on allocation failure.

## Notes

- `fwk`/`fws` are copied from the signed firmware footer — they identify *the running image*, not a per-device key. All devices running the same firmware report identical `fwk`/`fws`.
- `blv`/`blk`/`bls` are only emitted when the bootloader footer exists **and** its publisher matches the firmware publisher. Mismatch is silent (fields absent) rather than an error.
- `sd_csd`/`sd_cid` reveal SD-card hardware identifiers; they're gated behind auth after init to avoid leaking factory/model data to unauthenticated probes.

## Encedo Manager Usage

```javascript
app.api('api/system/version').then(function(data) {
    self.encedo_version_hardware = data.hwv;
    self.encedo_version_firmware = data.fwv;
    self.encedo_version_bootloader = data.blv;
    if (self.encedo_version_hardware.indexOf('EPA') > -1) {
        self.encedo_type = 'epa';
    }
});
```

Manager references: [encedo.js:196](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L196), [build.js:239](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L239), [build.js:4758](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L4758), [core2.js:2505](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2505)

## Test Suite Usage

```php
// hem-api-tester/test_0.php:27
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/version", $ret_val, $post_dummy, $token);
```

Test references: [test_0.php:27](https://github.com/encedo/hem-api-tester/blob/main/test_0.php#L27), [test_1.php:29](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L29), [test_7.php:69](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L69)
