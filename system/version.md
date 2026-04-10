# GET /api/system/version

Retrieve device hardware, firmware, and bootloader version information.

## Request

```
GET /api/system/version
```

**Authentication:** Not required (optional -- required only for microSD card CSD/CID values)

**Headers:** None required

## Response

**Status:** `200 OK`

```json
{
  "hwv": "PPA rev 2.2",
  "fwv": "Encedo nGINE FW v1.2.2",
  "fwk": "base64_encoded_firmware_signing_public_key",
  "fws": "base64_encoded_firmware_signature",
  "blv": "Encedo Secure Bootloader v2.0.1",
  "blk": "base64_encoded_bootloader_signing_public_key",
  "bls": "base64_encoded_bootloader_signature",
  "sd_csd": "microSD_CSD_value",
  "sd_cid": "microSD_CID_value",
  "uis": "manager_version_hash"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `hwv` | String | Hardware version (e.g., "PPA rev 2.2" or "EPA rev 1.0") |
| `fwv` | String | Firmware version |
| `fwk` | String | Firmware signing public key (base64) |
| `fws` | String | Firmware signature (base64) |
| `blv` | String | Bootloader version |
| `blk` | String | Bootloader signing public key (base64) |
| `bls` | String | Bootloader signature (base64) |
| `sd_csd` | String | Optional. microSD CSD value (requires valid token) |
| `sd_cid` | String | Optional. microSD CID value (requires valid token) |
| `uis` | String | Optional. Encedo Manager version hash |

## Encedo Manager Usage

```javascript
// encedo.js:196, build.js:239
app.api('api/system/version').then(function(data) {
    self.encedo_version_hardware = data.hwv;
    self.encedo_version_firmware = data.fwv;
    self.encedo_version_bootloader = data.blv;
    // Determines device type: PPA vs EPA
    if (self.encedo_version_hardware.indexOf('EPA') > -1) {
        self.encedo_type = 'epa';
    }
});
```

**Source references:** [encedo.js:196](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L196), [build.js:239](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L239), [build.js:4758](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L4758), [core2.js:2505](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2505)

## Test Suite Usage

```php
// hem-api-tester/test_0.php:27
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/version", $ret_val, $post_dummy, $token);
```

**Source references:** [test_0.php:27](https://github.com/encedo/hem-api-tester/blob/main/test_0.php#L27), [test_1.php:29](https://github.com/encedo/hem-api-tester/blob/main/test_1.php#L29), [test_7.php:69](https://github.com/encedo/hem-api-tester/blob/main/test_7.php#L69)

## Notes

- The `hwv` field is used to detect device type (PPA vs EPA) by checking for the "EPA" substring.
- The `uis` field is used by the dashboard to display the manager version.

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
