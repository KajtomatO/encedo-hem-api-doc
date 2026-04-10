# DELETE /api/logger/:id

Delete a specific audit log file.

## Request

```
DELETE /api/logger/:id
Authorization: Bearer <JWT_TOKEN>
```

**Scope:** `logger:del`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | String | Log file ID (hex string) |

## Response

**Status:** `200 OK`

### Error Codes

| Code | Description |
|------|-------------|
| 401 | Missing or invalid JWT token |
| 403 | Incorrect access scope |
| 404 | File not found |
| 406 | Operation failed |
| 409 | Incorrect internal state |

## Encedo Manager Usage

Registered in `_endpoints` map ([build.js:9183](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9183), [scopes.js:720](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L720)) but no direct DELETE call to a logger endpoint was found in the Manager UI JavaScript. The scope consent dialog for `logger:del` exists in `scopes.js`.

**Source references:** [build.js:9183](https://github.com/encedo/encedo-manager/blob/main/assets/build.js#L9183) (endpoint definition), [scopes.js:720](https://github.com/encedo/encedo-manager/blob/main/assets/scopes.js#L720) (scope definition)

## Official Docs vs Encedo Manager

> **DISCREPANCY:** The `_endpoints` map registers the scope as `logger:get` (not `logger:del`) for the DELETE method on `api/logger/*`. This appears to be an **error in [Encedo Manager](https://github.com/encedo/encedo-manager) endpoint registry** -- the scope consent system in `scopes.js` has a separate `logger:del` scope with "DELETE THE LOG" messaging. The actual scope sent in the JWT should be `logger:del`, but the endpoint map incorrectly lists `logger:get`.
