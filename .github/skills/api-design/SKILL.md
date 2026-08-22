---
name: api-design
description: Request/response contract and error-shape conventions for Route Handlers and Server Actions in RW-MVP. Use when designing a new API endpoint/action or changing an existing contract.
---

# API Design — RW-MVP

## Overview

Consistent contracts across `app/api/**/route.ts` and `lib/actions/` mean the Test agent and frontend code can rely on one shape everywhere. This skill governs request/response design, not the Next.js mechanics of writing the handler (see `nextjs-app-router`/`nextjs-server-actions` for that).

**Trigger:** designing a new endpoint or Server Action's contract, or changing an existing one's request/response shape.

## Error shape (mandatory, everywhere)

```json
{ "error": { "code": "VALIDATION_ERROR", "message": "title is required" } }
```

- `code` is a stable, machine-readable string (`UPPER_SNAKE_CASE`) the frontend can switch on — never change an existing `code` value once another part of the app depends on it.
- `message` is human-readable and safe to show a user — never leak a stack trace, SQL, or internal file path into it.
- Use HTTP status codes correctly on Route Handlers: `400` validation, `401` unauthenticated, `403` unauthorized, `404` not found, `409` conflict (e.g. unique constraint), `500` unexpected — don't return `200` with an error body.

## Success shape

- Return the resource/data directly (no unnecessary `{ data: ... }` wrapper) unless the endpoint returns a paginated list, in which case wrap as `{ items: [...], nextCursor: ... }` (or `{ items, total, page }` for offset pagination) — pick one pagination shape and use it project-wide.

## Request validation

- Every Route Handler and Server Action validates its input with zod before touching Prisma — reject early with `VALIDATION_ERROR` rather than letting a bad value reach the database and surface as a confusing Prisma error.
- Derive the zod schema's error messages into the `message` field directly — don't write a second, hand-maintained set of error strings.

## Contract stability

- Any change to a request/response shape, a `code` value, or a status code is a contract change — record it in `docs/knowledge/decisions/api.md` in the same change (per the project's DB-related-change workflow, contract changes get the same treatment).
- Additive changes (new optional field) are safe; removing/renaming a field or changing a `code`'s meaning is breaking — call it out explicitly and check for existing consumers before doing it.

## REST vs Server Action

- Use a Route Handler (`app/api/**/route.ts`) for anything called by an external client, webhook, or that needs a stable public contract.
- Use a Server Action (`lib/actions/`) for mutations triggered directly from the app's own forms/components — see `nextjs-server-actions` for its specific conventions.
- Don't implement the same mutation as both — pick one path per mutation.

## Common mistakes to catch

- Error response with no `{ error: { code, message } }` shape, or a `200` status on an error.
- `message` containing a stack trace, raw Prisma error, or internal detail.
- Two different pagination shapes used across different list endpoints.
- Contract change (field renamed/removed, `code` meaning changed) with no update to `docs/knowledge/decisions/api.md`.
- Same mutation implemented via both a Route Handler and a Server Action.
