# DELETE /api/logger/{id}

Delete a single audit log file.

## Firmware status — **not dispatched in current builds**

The handler function (`api_del_logger_file`) exists in the firmware source, but its dispatch in the HTTP router is wrapped in a `/* not available in CC mode */` comment block. As a result:

- The device **does not accept** `DELETE /api/logger/...` — every request falls through to the generic `404 Not Found`.
- This holds on both PPA and EPA builds shipped today.

The scope constant `logger:del` is still defined in the firmware header (`SCOPE_API_DEL_LOGGER`), so the endpoint can be re-enabled later simply by uncommenting the dispatch entry. Client tooling that advertises the scope is not broken — it will just get `404` until firmware re-enables the route.

## Intended behaviour (if re-enabled)

```
DELETE /api/logger/{id}
Authorization: Bearer <token>
```

- **Scope:** `logger:del`
- **Preconditions:** initialised, valid Bearer.
- **On success:** delete the file via `LOGGER_DeleteLogFile` and emit an audit record `LOG_TYPE_LOGGER_DELETE_LOG` (`LOG_RESULT_OK`).
- **On failure:** return `410` with a `LOG_RESULT_FAILED` audit record.

## Encedo Manager Usage

The endpoint is registered in the Manager's `_endpoints` map ([build.js:9183](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9183)) and the consent dialog for scope `logger:del` exists in [scopes.js:720](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L720), but no code path in the Manager UI actually calls `DELETE /api/logger/...`. The user-visible "delete log" flow in the Manager is disabled for the same reason the firmware disables it.

## Test Suite Usage

Not exercised by `hem-api-tester`.
