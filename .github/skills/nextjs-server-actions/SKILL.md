---
name: nextjs-server-actions
description: Conventions for writing and calling Server Actions in lib/actions/ for RW-MVP. Use when implementing any data mutation triggered from a form or client component (create/update/delete).
---

# Next.js Server Actions — RW-MVP

## Overview

Server Actions in `lib/actions/` are the mutation path for anything triggered by a form submit or a client-side interaction (as opposed to Route Handlers, which serve the `api-design` skill's REST contract for external/programmatic callers).

**Trigger:** implementing or editing a function in `lib/actions/`, or wiring a form/button to mutate data.

## File & function conventions

- Every Server Action file starts with `"use server"` at the top.
- One action per exported function; name it as a verb phrase (`createTask`, `updateTaskStatus`) — never a generic `handleSubmit`.
- Actions take a typed input (form data or a plain object) and return a typed result — never `any`.

## Validation & error shape

- Validate all incoming input with zod **inside the action**, even if the client already validated — the server is the trust boundary.
- On validation failure or business-rule violation, return `{ error: { code, message } }` (matching the project-wide API error shape) — don't `throw` raw errors across the server/client boundary for expected failures; reserve `throw` for truly unexpected/programmer errors.
- Never expose Prisma error internals (constraint names, stack traces) to the client — map known Prisma error codes (e.g. `P2002` unique violation) to a domain-meaningful `{ code, message }`.

## Cache invalidation

- After any mutation that changes data a Server Component reads, call `revalidatePath(path)` or `revalidateTag(tag)` before returning — a mutation with no revalidation call is a bug (stale UI), flag it in review.
- Prefer `revalidateTag` when multiple routes read the same data; use `revalidatePath` for a single, obviously-scoped page.

## Calling from the client

- Use the action directly as a form `action` prop where possible (progressive enhancement, no client JS required for the base case).
- When client-side state is needed around the call (pending/error UI), wrap with `useTransition` (`isPending`) rather than manual `useState` loading flags.
- Never call `fetch('/api/...')` from a Client Component when a Server Action already exists for that mutation — pick one path per mutation, don't duplicate logic across both.

## Common mistakes to catch

- Missing `"use server"` directive.
- No zod validation on the action's input.
- Mutation with no `revalidatePath`/`revalidateTag` call.
- Prisma error object (or its message) returned to the client as-is.
- Same mutation implemented twice — once as a Server Action, once as a Route Handler — with logic drifting between them.
