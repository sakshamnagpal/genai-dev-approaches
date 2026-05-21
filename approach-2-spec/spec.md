# User Listing API — Specification

## Overview

An internal REST API endpoint for an admin tool that returns a paginated, filterable list of users. This spec defines the interface contract, validation rules, error handling behaviour, and governance requirements that must be satisfied before implementation is considered complete.

---

## Endpoint

`GET /users`

---

## Query Parameters

| Parameter   | Type    | Required | Default        | Description                    |
|-------------|---------|----------|----------------|--------------------------------|
| `page`      | integer | No       | `1`            | Page number, 1-indexed         |
| `page_size` | integer | No       | `20`           | Number of results per page     |
| `status`    | string  | No       | none (all)     | Filter by user status          |

---

## Response Contract

### Success — 200

```json
{
  "users": [
    {
      "id": "string",
      "name": "string",
      "email": "string",
      "status": "string",
      "created_at": "ISO 8601 string"
    }
  ],
  "total": 84,
  "page": 1,
  "page_size": 20
}
```

### Error — 400

```json
{
  "error": "string"
}
```

---

## Validation Rules

- `page` must be a positive integer (≥ 1)
- `page_size` must be a positive integer between 1 and 100 inclusive
- `status` must be one of: `active`, `inactive`, `pending`
- If `page` exceeds available results, return an empty `users` array — not an error
- Missing parameters fall back to defaults silently

---

## Data Model

For this example, the data store is an in-memory list of users. Each user object has:

- `id` — unique string identifier
- `name` — full name string
- `email` — email address string
- `status` — one of `active`, `inactive`, `pending`
- `created_at` — ISO 8601 datetime string

---

## Error Handling Rules

- Invalid parameter values return `400` with a descriptive `error` message
- Error messages must describe what is wrong without leaking implementation details (no stack traces, no internal variable names)
- All errors return JSON, never plain text

---

## Governance & Security Requirements

These are non-negotiable and must be satisfied in all phases:

- **No PII in logs** — user names, emails, and IDs must never appear in log output
- **Input sanitisation** — all query parameters must be validated and typed before use; never pass raw query string values to the data layer
- **Dependency constraints** — implementation must use Python stdlib only; no third-party packages
- **Error message hygiene** — error responses must not reveal internal implementation details, file paths, or stack traces
- **No silent failures** — validation errors must always return a `400`; the API must never return a `200` with empty or unexpected data due to a bad input being silently ignored

---

## Out of Scope

- Authentication and authorisation
- A real database (in-memory data store only)
- Rate limiting implementation (must be noted as a future consideration in code comments)
- Deployment configuration