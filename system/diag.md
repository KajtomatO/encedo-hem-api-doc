# /api/diag/* — Diagnostic Endpoints

> **Do not call these on a production device.** The handlers here destroy persistent state, corrupt the key repository, downgrade TLS, and otherwise push the device into failure modes for hardware-validation / QA use.

Compiled into the firmware only when `DIAG_MODE_ON` is defined. Production PPA/EPA builds return `404` for every path under `/api/diag/`.

## Common traits

- All endpoints use `GET`.
- **No Bearer token is required for any handler here.** They pre-date and bypass the normal JWT dispatcher.
- A trailing `/test` handler (`GET /api/diag/test`) simply returns `200` — a liveness probe confirming DIAG mode is compiled in.

## Endpoints

| Endpoint | Effect on device | Response |
|----------|------------------|----------|
| `GET /api/diag/test` | Liveness probe — confirms this is a DIAG build. | `200` |
| `GET /api/diag/test_trng` | Draws 8192 bytes from the TRNG and returns them base64-encoded. Used for entropy analysis. | `200 {"rnd":"<base64>"}` or `400`/`406` |
| `GET /api/diag/break_trng` | Toggles `Session.diag_trng = 1` (fault-injection flag that makes the next entropy self-test fail). Returns the previous mode and last entropy-test stats. | `200 {"mode":…,"last_ts":…,"last_val":…}` |
| `GET /api/diag/break_tls` | Sets `Session.diag_tls = 5`, causing the next 5 TLS transactions to inject an error at the TCP/IP layer. | `200` |
| `GET /api/diag/break_temp` | Sets `Session.diag_temp = 1` (simulates temperature-sensor failure). | `200` |
| `GET /api/diag/disable_selftest` | Sets `Session.disable_selftest = 1` — subsequent `GET /api/system/selftest` short-circuits. | `200` |
| `GET /api/diag/wipe_config` | Wipes all persistent config, formats the microSD (PPA), reboots into the bootloader. Closes the connection immediately before running. | `200` + reboot |
| `GET /api/diag/corrupt_repo/<address>` | Writes 4 bytes of zeros to FLASH at `<address>+4`. Address must be within the key-repo range, chunk-aligned. | `200 {"address":…,"status":0\|1}` or `400` |
| `GET /api/diag/memdump/<address>/<length>` | Base64-dumps `<length>` bytes of memory starting at `<address>`. Length capped at 4096. | `200 {"address":…,"length":…,"dump":"<base64>"}` |

## Notes

- `memdump` and `corrupt_repo` read/write **any** absolute memory address within the bounds checked (repo range for corrupt, arbitrary for memdump). On a DIAG build this is exposed without auth, so DIAG firmware must never be shipped to users.
- `break_trng` / `break_tls` / `break_temp` state persists only until reboot or until the injected fault counter expires (`Session.diag_tls` counts down on each TLS transaction).
- Test-suite gating (`strstr($fwv, "-DIAG")`) is a convention — the DIAG firmware's `fwv` contains the literal substring `-DIAG`.

## Encodo Manager Usage

Not present in the Manager.

## Test Suite Usage

The `hem-api-tester` suite exercises these endpoints in `test_8.php` (security-test scenarios) and occasionally in other tests that need to force a particular failure mode.

Test references: [test_8.php:126](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L126), [test_8.php:189](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L189), [test_8.php:242](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L242), [test_8.php:315](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L315), [test_8.php:322](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L322), [test_8.php:357](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L357), [test_8.php:416](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L416)
