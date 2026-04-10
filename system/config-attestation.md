# GET /api/system/config/attestation

Retrieve device attestation data (CSR and genuine token). Encedo PPA only.

## Request

```
GET /api/system/config/attestation
Authorization: Bearer <JWT_TOKEN>  (optional on fresh devices)
```

**Authentication:** Optional on fresh, unpersonalized devices. Ignores access scope.

**Availability:** Encedo PPA only

## Response

**Status:** `200 OK`

```json
{
  "crt": "base64_encoded_der_x509_device_certificate",
  "genuine": "attestation_data"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `crt` | String | Base64 encoded DER x509 device certificate |
| `genuine` | String | Attestation data |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 409 | Incorrect internal state |

## Encedo Manager Usage

```javascript
// Used in provisioning flow and rescue mode
// encedo.js:654, build.js:798, core-rescue.js:763, core-rescue-final.js:833
xhr.open('GET', self.encedo_url + '/api/system/config/attestation');
```

**Source references:** [encedo.js:654](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L654), [build.js:798](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L798), core-rescue.js:763, core-rescue-final.js:833

## Test Suite Usage

```php
// hem-api-tester/test_4.php:171
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/config/attestation", $ret_val, $post_dummy, $token);
```

**Source references:** [test_4.php:171](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L171), [test_4.php:176](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L176)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
