# GET /api/system/config/attestation

Return ATECC608 attestation material — either a **CSR** (when the secure element has no device cert yet) or the **stored cert** (when it does). In both cases, an attestation token (`genuine`) proving the ATECC608 is a real Encedo device is returned.

**PPA only.** (Compiled behind `USB_MSC_AVAILABLE`.)

## Request

```
GET /api/system/config/attestation
Authorization: Bearer <token>   # only required once initialised
```

**Authentication:**
- On an **initialised** device: Bearer required. No scope check — any valid token works. Rationale in firmware: "Provisioning is possible only during production first run and only once. If the unit is initialized, valid JWT is required to limit tracking."
- On a **fresh/uninitialised** device: no auth.
- Software install in progress → `409`.
- `fls_state != 0` → `409` (self-test must be clean).

## Response

**Status:** `200 OK`

Two shapes depending on ATECC608 state:

**No cert yet (first boot / provisioning path):**

```json
{
  "csr": "PEM_CSR_generated_by_atecc608a_gen_csr",
  "key": "base64_der_attestation_public_key",
  "genuine": "atecc608_attestation_token"
}
```

**Cert already written (normal path):**

```json
{
  "crt": "base64_der_x509_device_certificate",
  "genuine": "atecc608_attestation_token"
}
```

| Field | Type | When emitted | Description |
|-------|------|--------------|-------------|
| `csr` | String | Only when ATECC608 slot has no cert | PEM-format CSR. Ship to Encedo CA to be signed. |
| `key` | String | Same condition as `csr` | Base64 DER of the ATECC608 attestation public key (slot 1). |
| `crt` | String | Only when ATECC608 slot already has a cert | Base64 DER of the stored x509 device certificate. |
| `genuine` | String | Always | Attestation token over timestamp `0` from slot 0, proving the private key is held by a real ATECC608. |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Initialised device without Bearer. |
| 409 | Software install in progress, or `fls_state != 0`. |
| 500 | ATECC608 communication failure (body: `"atecc_1"` … `"atecc_4"`). |

## Notes

- The "fresh chip" path generates a new CSR every call — the ATECC608 key isn't regenerated, only the CSR encoding is.
- `genuine` is always fresh because it includes a newly-timestamped attestation token.

## Encedo Manager Usage

```javascript
// encedo.js:654, build.js:798 — provisioning / rescue paths
xhr.open('GET', self.encedo_url + '/api/system/config/attestation');
```

Manager references: [encedo.js:654](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L654), [build.js:798](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L798)

## Test Suite Usage

```php
// test_4.php:171
http_transaction("https", "GET", $cfg_domain, "/api/system/config/attestation", $ret_val, $post_dummy, $token);
```

Test references: [test_4.php:171](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L171)
