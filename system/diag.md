# /api/diag/* -- Diagnostic Endpoints

> **CAUTION:** These endpoints perform destructive operations including disabling TLS, corrupting the key repository, wiping device configuration, and simulating hardware failures. Using them can render the device inoperable or require factory reset. Use only in controlled test environments with non-production devices.

Fault injection and diagnostic endpoints available **only on diagnostic firmware builds** (firmware version contains `-DIAG` suffix). These are used for hardware validation and security testing, not for production use.

## Availability

These endpoints are only available when the device runs a diagnostic firmware build. The test suite checks `strstr($fwv, "-DIAG")` before calling them. On production firmware, these endpoints do not exist.

## Endpoints

All endpoints use `GET` method. Authentication is **not required** for some (e.g., `break_tls` uses HTTP).

| Endpoint | Description | Protocol | Source |
|----------|-------------|----------|--------|
| `GET /api/diag/break_tls` | Simulate TLS failure / disable TLS enforcement | HTTP | [test_8.php:126](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L126) |
| `GET /api/diag/break_trng` | Toggle TRNG (True Random Number Generator) failure simulation | HTTPS | [test_8.php:189](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L189), [test_8.php:203](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L203) |
| `GET /api/diag/test_trng` | Run TRNG diagnostic test | HTTPS | [test_8.php:242](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L242) |
| `GET /api/diag/memdump/:addr/:len` | Dump memory contents at given address and length | HTTPS | [test_8.php:315](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L315), [test_8.php:327](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L327) |
| `GET /api/diag/corrupt_repo/:addr` | Corrupt the key repository at given memory address | HTTPS | [test_8.php:322](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L322) |
| `GET /api/diag/wipe_config` | Wipe device configuration (factory reset) | HTTPS | [test_8.php:357](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L357) |
| `GET /api/diag/break_temp` | Simulate temperature sensor failure | HTTP/HTTPS | [test_8.php:416](https://github.com/encedo/hem-api-tester/blob/main/test_8.php#L416), [test_11.php:784](https://github.com/encedo/hem-api-tester/blob/main/test_11.php#L784), [test_5.php:185](https://github.com/encedo/hem-api-tester/blob/main/test_5.php#L185) |

## Encedo Manager Usage

**Not present in the Manager UI.** None of these endpoints appear in `_endpoints`, `scopes.js`, or any [Encedo Manager](https://github.com/encedo/encedo-manager) JavaScript. They exist only in the [HEM API test suite](https://github.com/encedo/hem-api-tester) for hardware validation.

## Official Docs vs Encedo Manager

> **Not documented** in official docs. Diagnostic-firmware-only endpoints discovered in the [HEM API test suite](https://github.com/encedo/hem-api-tester) (`test_8.php`). These are development/QA tools, not intended for end-user applications.
