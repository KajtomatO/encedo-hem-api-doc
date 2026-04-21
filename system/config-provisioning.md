# POST /api/system/config/provisioning

Write a CA-signed device certificate into the ATECC608 secure element. One-shot factory endpoint: it refuses to run if the device is initialised **or** if a cert is already stored in the ATECC608 slot.

**PPA only** (compiled behind `USB_MSC_AVAILABLE`).

## Request

```
POST /api/system/config/provisioning
Content-Type: application/json
```

**Authentication:** Not required. Access control is entirely state-based:
- `is_initialised() == true` → `403` (the unit has already passed provisioning and is paired to a user).
- ATECC608 slot 0 already contains a DER cert → `406`.

### Request Body

```json
{
  "crt": "base64_der_signed_certificate",
  "genuine": "attestation_token_from_previous_GET"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `crt` | String | Yes | Base64 DER of the signed x509 certificate from the Encedo CA. |
| `genuine` | String | Yes | Attestation token from the prior `GET /api/system/config/attestation` call. The device re-validates this token against the ATECC608 before trusting the cert. |

### Validation pipeline

1. Parse JSON; `400` on failure.
2. ATECC608 slot 0 must be empty; `406` otherwise.
3. Validate `genuine` via `atecc608_attestation_token_validate` → `400` on failure.
4. Base64-decode `crt` → `400` on failure.
5. Verify the DER bytes **contain** the ATECC608 slot-0 public key (substring match via `memmem` — not a structured x509 parse). Missing → `406`.
6. Write DER to ATECC608 via `atecc608_write_cert_der`. Failure after one retry → `406`.

### Response

`200 OK` on success. No body.

### Error Codes

| Code | Description |
|------|-------------|
| 400 | Malformed JSON, missing/invalid `genuine`, malformed `crt`. |
| 403 | Device already initialised. |
| 406 | Cert already written, or cert body does not contain the ATECC608 pubkey, or ATECC608 write failed. |
| 500 | ATECC608 read failure / allocation failure. |

## Notes

- The pubkey presence check (step 5) is a **byte-substring match** rather than a full certificate parse; an attacker-supplied cert that happens to contain the pubkey bytes in any field would pass. However this handler is only reachable on factory-fresh devices before auth is set up, so the practical threat model is limited.
- The corresponding GET (`/api/system/config/attestation`) must be called **before** this POST to produce a fresh `genuine` token.

## Encedo Manager Usage

```javascript
// encedo.js:678, build.js:822
xhr.open('POST', self.encedo_url + '/api/system/config/provisioning');
xhr.send(JSON.stringify(cert));
```

Manager references: [encedo.js:678](https://github.com/encedo/encedo-manager/blob/main/assets/encedo.js#L678), [build.js:822](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L822)

## Test Suite Usage

```php
// test_4.php:182
http_transaction("https", "POST", $cfg_domain, "/api/system/config/provisioning", $ret_val, $post_data, $token);
```

Test references: [test_4.php:182](https://github.com/encedo/hem-api-tester/blob/main/test_4.php#L182)
