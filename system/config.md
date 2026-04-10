# /api/system/config

Read and update the device configuration.

---

## GET /api/system/config

Retrieve the current device configuration.

### Request

```
GET /api/system/config
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `system:config`

### Response

**Status:** `200 OK`

```json
{
  "iat": 1647936471,
  "uts": 1647936471,
  "devid": "device_unique_id",
  "instanceid": "e7aafede-76b1-c7c8-784b-c963967fe307",
  "eid": "base64_encoded_encedo_id",
  "eid_sign": "base64_encoded_audit_log_signing_key",
  "user": "username",
  "email": "user@example.com",
  "hostname": "my.ence.do",
  "dnsd": true,
  "trusted_ts": true,
  "trusted_backend": true,
  "allow_keysearch": true,
  "origin": "*",
  "ctx": 1,
  "http_option_hsts": true,
  "http_option_dosprot_mode": 10,
  "ip": "192.168.1.1/24",
  "genuine_id": "secure_enclave_serial",
  "storage_mode": 1,
  "storage_disk0size": 1024,
  "storage_capacity": 2048,
  "spk": "base64_session_public_key",
  "nonce": "base64_random_nonce"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `iat` | Number | Current timestamp |
| `uts` | Number | Last RTC update timestamp |
| `devid` | String | Device unique ID |
| `instanceid` | String | Instance unique ID |
| `eid` | String | EncedoID -- public key of instance |
| `eid_sign` | String | Audit log signing public key |
| `user` | String | Username / display name |
| `email` | String | Email address |
| `hostname` | String | Hostname / domain name |
| `dnsd` | Boolean | DNS server enabled |
| `trusted_ts` | Boolean | Backend trusted as time source |
| `trusted_backend` | Boolean | Backend trusted to control instance |
| `allow_keysearch` | Boolean | Allow unauthenticated key search |
| `origin` | String | CORS allowed origins |
| `ctx` | Number | Instance context ID |
| `http_option_hsts` | Boolean | HSTS headers enabled |
| `http_option_dosprot_mode` | Number | HTTP request timeout (0.5sec units) |
| `ip` | String | Device IP address/CIDR |
| `genuine_id` | String | Secure Enclave serial number |
| `storage_mode` | Number | Disk encryption mode |
| `storage_disk0size` | Number | Disk 0 capacity in sectors |
| `storage_capacity` | Number | MicroSD total capacity in sectors |
| `spk` | String | Session public key |
| `nonce` | String | Random nonce |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 400 | Incorrect argument(s) |
| 406 | Operation failed |
| 409 | Incorrect internal state |

---

## POST /api/system/config

Update the device configuration. Only provided fields are updated.

### Request

```
POST /api/system/config
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

**Scope:** `system:config`

### Request Body

All fields are optional -- only include fields to update:

```json
{
  "allow_keysearch": true,
  "email": "new@example.com",
  "storage_mode": 1,
  "wipeout": true,
  "origin": "*",
  "trusted_ts": true,
  "trusted_backend": true,
  "tls": "x509_certificate_data",
  "user": "newuser",
  "userkey": "new_password_public_key",
  "userkey_nonce": "password_auth_nonce",
  "userkey_hmac": "password_auth_code",
  "ctx": 1,
  "dnsd": true,
  "storage_disk0size": 1024,
  "gen_csr": true,
  "emp": "ephemeral_public_key",
  "key": "encrypted_private_key",
  "crt": "base64_der_x509_certificate",
  "http_option_hsts": true,
  "http_option_dosprot_mode": 10
}
```

| Field | Type | Description |
|-------|------|-------------|
| `allow_keysearch` | Boolean | Allow unauthenticated key search |
| `email` | String | Contact email |
| `storage_mode` | Number | Disk encryption mode |
| `wipeout` | Boolean | **Reset to factory defaults** |
| `origin` | String | CORS access control |
| `trusted_ts` | Boolean | Backend trusted as time source |
| `trusted_backend` | Boolean | Backend trusted to control instance |
| `tls` | String | TLS x509 certificate data |
| `user` | String | Username |
| `userkey` | String | New password public key |
| `userkey_nonce` | String | Password auth nonce |
| `userkey_hmac` | String | Password auth HMAC |
| `ctx` | Number | Instance context ID |
| `dnsd` | Boolean | Enable DNS server |
| `storage_disk0size` | Number | Disk 0 size in sectors |
| `gen_csr` | Boolean | Request CSR generation |
| `emp` | String | Ephemeral public key |
| `key` | String | Encrypted private key |
| `crt` | String | Base64 encoded DER x509 certificate |
| `http_option_hsts` | Boolean | Enable HSTS headers |
| `http_option_dosprot_mode` | Number | HTTP timeout (0.5sec units) |

### Response

**Status:** `200 OK`

```json
{
  "csr": "base64_encoded_der_csr",
  "genuine": "attestation_data",
  "updated": true,
  "reboot_required": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| `csr` | String | Base64 encoded DER CSR (if `gen_csr` was true) |
| `genuine` | String | Attestation data |
| `updated` | Boolean | Whether configuration was changed |
| `reboot_required` | Boolean | Whether a reboot is needed |

## Encedo Manager Usage

```javascript
// Factory reset (wipeout)
// build.js:1861
app.api('api/system/config', 'POST', { wipeout: true });

// TLS certificate update
// build.js:4277, core2.js:2024
app.api('api/system/config', 'POST', { tls: certData });

// Password change
// build.js:7038
app.api('api/system/config', 'POST', {
    userkey: newPublicKey,
    userkey_nonce: nonce,
    userkey_hmac: hmac
});

// Settings update
// build.js:7081-7241
app.api('api/system/config', 'POST', { user: name, email: email, ... });
```

**Source references:** [encedo.js:242](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L242), [build.js:285](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L285), [build.js:1331](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1331), [build.js:1861](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L1861), [build.js:4241](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L4241), [build.js:6887](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L6887), [core2.js:1988](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L1988), [core2.js:4592](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L4592)

## Test Suite Usage

```php
// hem-api-tester/test_4.php:64 (GET)
$ret_stat = http_transaction("https", "GET", $cfg_domain, "/api/system/config", $ret_val, $post_dummy, $token);

// hem-api-tester/test_2.php:187 (POST)
$ret_stat = http_transaction("https", "POST", $cfg_domain, "/api/system/config", $ret_val, $post_data, $token);
```

**Source references:** [test_2.php:187](https://github.com/encedo/hem-api-tester/blob/main/test_2.php#L187), [test_4.php:64](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L64), [test_4.php:72](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L72)

## Official Docs vs Encedo Manager

**Status:** Documented -- matches [Encedo Manager](https://github.com/encedo/encedo-manager). The official docs are thorough for this endpoint.

> **DISCREPANCY:** The official docs state `Authorization` is required for GET. However, [Encedo Manager](https://github.com/encedo/encedo-manager) also calls `GET /api/system/config` **without auth** during pairing flows ([build.js:383](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L383), [build.js:419](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L419), [build.js:455](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L455)) to read `eid`. The device appears to return at least `eid` without auth, though a full config read requires a token.
