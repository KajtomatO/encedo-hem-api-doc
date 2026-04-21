# GET /api/logger/key

Return the device's **audit-log signing public key** (Ed25519) together with a fresh nonce and the nonce signed by the matching private key. The caller uses these to verify the Ed25519 signatures embedded in every audit-log entry.

Available on both PPA and EPA.

## Request

```
GET /api/logger/key
Authorization: Bearer <token>
```

**Authentication:** Bearer required. Scope must start with `logger:get`.

**Preconditions:** Initialised — otherwise `409`.

## Response

**Status:** `200 OK`

```json
{
  "key":          "base64_ed25519_public_key",
  "nonce":        "base64_32_byte_nonce",
  "nonce_signed": "base64_ed25519_signature"
}
```

| Field | Description |
|-------|-------------|
| `key` | `Session.EIDkeySign` public half — Ed25519 32-byte key derived at boot from `DeviceKey`. The same key signs every `LOG_TYPE_*` entry in audit logs. |
| `nonce` | 32-byte time-based nonce (base64). |
| `nonce_signed` | Ed25519 signature over `nonce` (base64). Confirms the caller is talking to the device that holds the private half. |

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing/invalid Bearer. |
| 403 | Wrong scope. |
| 409 | Not initialised. |
| 500 | Allocation failure. |

## Notes

- The key is **per-device, not per-firmware**: `EIDkeySign` is re-derived from the device master secret at every boot but yields the same public key. Wipeout + re-init rotates it.
- This endpoint is sometimes confused with [`/api/auth/init → eid`](../auth/init.md) — both return public keys, but:
  - `eid` is `Session.EIDkey` (Curve25519, used for ECDH).
  - `/api/logger/key → key` is `Session.EIDkeySign` (Ed25519, used for signatures on audit logs).

## Encedo Manager Usage

```javascript
// build.js:5217, core2.js:2955
app.api('api/logger/key').then(function(signature) {
    // verify {signature.key, signature.nonce, signature.nonce_signed}
});
```

Manager references: [build.js:5217](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L5217), [core2.js:2955](https://github.com/encedo/encedo-manager/blob/main/assets/core2.js#L2955)

## Test Suite Usage

```php
// test_3.php:174, test_9.php:62
http_transaction("https", "GET", $cfg_domain, "/api/logger/key", $ret_val, "", $token);
```

Test references: [test_3.php:174](https://github.com/encedo/hem-api-tester/blob/main/test_3.php#L174), [test_9.php:62](https://github.com/encedo/hem-api-tester/blob/main/test_9.php#L62)
