# SimpleMem HTTP API

REST API for user and auth management. Used by the SimpleMem MCP server for registration, token management, and account deletion.

**Base URL (self-hosted):** `http://localhost:8000`  
**Base URL (cloud):** `https://mcp.simplemem.cloud`

Protected endpoints require the `Authorization: Bearer <token>` header. Obtain a token via the create user (register) endpoint.

---

## Authentication

Endpoints that require authentication expect:

```
Authorization: Bearer <your-token>
```

The token is a JWT returned when you create a user. Use it for delete user, verify, refresh, and all MCP requests.

---

## Create user

Creates a new user and returns an API token and user ID. This is the main way to get a token for MCP and other authenticated APIs.

**Endpoint:** `POST /api/auth/register`

**Request body:**

```json
{
  "openrouter_api_key": "sk-or-v1-..."
}
```

- **openrouter_api_key** (required for OpenRouter): Your OpenRouter API key. It is validated before the user is created.
- When using **Ollama** as the LLM provider, the key can be empty or a placeholder (e.g. `"ollama-placeholder-key"`); the server will still validate that Ollama is reachable.

**Success response:** `200 OK`

```json
{
  "success": true,
  "token": "<jwt>",
  "user_id": "<uuid>",
  "mcp_endpoint": "/mcp"
}
```

- **token:** JWT to use in `Authorization: Bearer <token>`.
- **user_id:** Unique user identifier; store it if you need it for delete user or other flows.
- **mcp_endpoint:** Path to the MCP endpoint (relative to base URL).

**Error response:** `200 OK` with `success: false`

```json
{
  "success": false,
  "error": "Invalid OpenRouter API key"
}
```

Common errors: invalid or unreachable OpenRouter API key; when using Ollama, inability to connect to Ollama.

---

## Delete user

Deletes the authenticated user's account. Only the token owner can delete their own user (token `user_id` must match the path `user_id`). All user data (metadata and memory table) is removed.

**Endpoint:** `DELETE /api/users/{user_id}`

**Headers:**

- `Authorization: Bearer <token>` (required)

**Path parameters:**

- **user_id:** The user ID to delete (must match the user identified by the token).

**Success response:** `204 No Content`

No body.

**Error responses:**

- **401 Unauthorized:** Missing or invalid token (e.g. expired, malformed).
- **403 Forbidden:** Token is valid but the token’s user does not match `user_id` (cannot delete another user).
- **404 Not Found:** User not found (e.g. already deleted).

---

## Verify token

Checks that a token is valid and returns the associated user ID.

**Endpoint:** `GET /api/auth/verify?token=<token>`

**Query parameters:**

- **token:** The JWT to verify.

**Success response:** `200 OK`

```json
{
  "valid": true,
  "user_id": "<uuid>"
}
```

**Error response:** `401 Unauthorized` if the token is invalid or expired.

---

## Refresh token

Issues a new token for the same user, with a new expiration. Use when the current token is close to expiring.

**Endpoint:** `POST /api/auth/refresh?token=<token>`

**Query parameters:**

- **token:** Current valid JWT.

**Success response:** `200 OK`

```json
{
  "success": true,
  "token": "<new-jwt>"
}
```

**Error responses:** `401 Unauthorized` if the token is invalid or expired; `404 Not Found` if the user no longer exists.

---

## Health and server info

**GET /api/health** — Health check. Returns `200 OK` with status and timestamp.

**GET /api/server/info** — Server information (version, models, session count, etc.). No authentication required.
