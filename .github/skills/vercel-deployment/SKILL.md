---
name: vercel-deployment
description: Environment variables, runtime selection, and deployment conventions for hosting RW-MVP on Vercel. Use when touching env vars, vercel.json, a Route Handler's runtime config, or anything CI/deploy related.
---

# Vercel Deployment — RW-MVP

## Overview

RW-MVP deploys to Vercel. This skill covers the environment-variable, runtime, and connection-pooling decisions that only matter once code leaves `localhost`.

**Trigger:** adding/changing an environment variable, setting a route's `runtime` export, editing `vercel.json`, or discussing deploy/CI behavior.

## Environment variables

- Never hardcode a secret, API key, or connection string in source — always read from `process.env`, set locally via `.env` (never committed) and in production via Vercel Project Settings.
- Every new environment variable used in code must be documented (name + purpose) so the next person can set it in Vercel — don't leave it discoverable only by reading the code.
- Distinguish `NEXT_PUBLIC_*` (exposed to the browser bundle) from server-only vars — never prefix a secret with `NEXT_PUBLIC_`.

## Runtime selection

- Route Handlers and Server Actions default to the **Node.js runtime** — only opt into the Edge runtime (`export const runtime = 'edge'`) when a route has no Prisma/Node-only dependency and actually benefits from edge latency.
- Verify Prisma compatibility before ever setting `runtime = 'edge'` on a route that touches the database — Prisma's standard client requires Node.js (a Data Proxy/Accelerate setup is a separate, deliberate decision, not a default).

## Database connections in serverless

- Confirm the production `DATABASE_URL` goes through a connection pooler (PgBouncer, Prisma Accelerate, or the provider's pooled connection string) — direct connections exhaust Postgres's connection limit under concurrent serverless invocations.
- Don't assume a global variable persists connection state reliably across invocations in production the way it appears to in local dev.

## Build & deploy

- `npm run build` must pass locally before pushing — Vercel's build failing in CI after a broken local state wastes a deploy cycle.
- Any new dependency needed only at build/deploy time (not runtime) belongs in `devDependencies`.
- Migrations run via `prisma migrate deploy` as an explicit release step (not `migrate dev`) — confirm how/when this runs relative to the deploy (pre-deploy hook, manual step, or CI job) rather than assuming it happens automatically.

## Common mistakes to catch

- Secret committed in `.env` or hardcoded in source.
- Secret accidentally prefixed with `NEXT_PUBLIC_`.
- `runtime = 'edge'` set on a route that imports Prisma without Data Proxy/Accelerate.
- Non-pooled `DATABASE_URL` used in the production environment variable.
- `prisma migrate dev` referenced anywhere in a CI/deploy script (should be `migrate deploy`).
