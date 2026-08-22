---
name: prisma-postgres
description: Conventions for Prisma schema design, migrations, and query patterns against PostgreSQL for RW-MVP. Use when editing prisma/schema.prisma or writing any Prisma query.
---

# Prisma + PostgreSQL — RW-MVP

## Overview

`prisma/schema.prisma` is the single source of truth for the data model; `lib/db.ts` exports the one shared `PrismaClient` instance. This skill covers schema changes, migrations, and query-writing conventions.

**Trigger:** editing `prisma/schema.prisma`, running a migration, or writing/reviewing a Prisma query.

## Schema conventions

- Model names: PascalCase singular (`Task`, not `tasks`). Field names: camelCase.
- Every model gets an `id` (prefer `String @id @default(cuid())` unless the domain needs a natural key), `createdAt DateTime @default(now())`, and `updatedAt DateTime @updatedAt`.
- Every foreign key column gets an explicit `@relation` and an `@@index` (or is already indexed via `@unique`) — an FK with no index is a performance bug caught at review, not just at scale.
- Use `enum` for a closed set of string values (status fields) instead of a free-text `String`.
- Add `@@unique([...])` for any natural-key uniqueness constraint instead of enforcing it only in application code.

## Migrations

- Every schema change: `npx prisma migrate dev --name <short-description>` locally — never hand-edit a file already committed under `prisma/migrations/`.
- One migration per logical schema change; don't bundle an unrelated model change into the same migration.
- After a migration, update `docs/knowledge/decisions/domain.md` in the same change if the business model shifted (new field meaning, new relation, new constraint).
- `prisma migrate deploy` is the only command that runs in CI/production — `migrate dev` is local-only (it can reset/interactively prompt).

## Query patterns

- Always go through `lib/db.ts`'s singleton — `new PrismaClient()` anywhere else exhausts Postgres connections under Vercel's serverless model.
- Use `select`/`include` deliberately — don't fetch a full relation when only a few fields or a `_count` are needed.
- Batch related fetches with a single `include`/`select` instead of looping and querying per row (see `postgres-performance` skill for the N+1 pattern this avoids).
- Use `$transaction` for any multi-step write that must succeed or fail atomically (e.g. create-parent-then-children).
- Never build `$queryRaw`/`$executeRaw` with string concatenation of user input — always the tagged-template/parameterized form.

## Error handling

- Catch Prisma's typed errors (`Prisma.PrismaClientKnownRequestError`) and map known codes (`P2002` unique constraint, `P2025` record not found) to the project's `{ error: { code, message } }` shape — don't let a raw Prisma error reach an API response.

## Common mistakes to catch

- `new PrismaClient()` instantiated outside `lib/db.ts`.
- New foreign key column with no `@@index`.
- Hand-edited file under `prisma/migrations/`.
- Schema change with no corresponding migration committed.
- `domain.md` not updated after a business-model-changing migration.
- Raw string interpolation into `$queryRaw`.
