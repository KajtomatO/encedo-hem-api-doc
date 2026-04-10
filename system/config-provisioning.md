# POST /api/system/config/provisioning

Factory provisioning endpoint. Imports a signed certificate and attestation data. Available only during manufacturing; disabled after first successful use.

## Request

```
POST /api/system/config/provisioning
Content-Type: application/json
```

**Authentication:** Not required (manufacturing only)

**Availability:** Encedo PPA only. Disabled after first successful provisioning.

### Request Body

```json
{
  "crt": "x509_certificate",
  "genuine": "attestation_data"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `crt` | String | Yes | x509 certificate |
| `genuine` | String | Yes | Attestation data |

## Response

**Status:** `200 OK` -- Successful provisioning

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed arguments |
| 403 | Device already personalized |
| 406 | Operation failed |

## Encedo Manager Usage

```javascript
// encedo.js:678, build.js:822
// Part of the provisioning() method
xhr.open('POST', url + '/api/system/config/provisioning');
xhr.send(JSON.stringify(cert));
```

**Source references:** [encedo.js:678](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L678), [build.js:822](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L822)

## Test Suite Usage

```php
// hem-api-tester/test_4.php:182 (POST)
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/system/config/provisioning", $ret_val, $post_data, $token);
```

**Source references:** [test_4.php:182](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L182)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager).
