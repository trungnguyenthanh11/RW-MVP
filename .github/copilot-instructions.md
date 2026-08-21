# RW-MVP — Copilot Instructions

## Tech stack

- Framework: Next.js (App Router), TypeScript
- Database: PostgreSQL, accessed via Prisma ORM
- Styling: Tailwind CSS (adjust if different)
- Testing: unit/integration (Vitest/Jest — fill in actual tool) + Playwright for E2E
- Deploy: Vercel

## Folder structure

- `app/` — routes, layouts, server components, API routes (`app/api/**/route.ts`)
- `lib/` — shared helpers (`lib/db.ts` Prisma singleton, `lib/actions/` Server Actions, auth, validation)
- `prisma/schema.prisma` + `prisma/migrations/` — DB schema and migration history
- `components/` — shared UI components
- `e2e/` — Playwright tests
- `docs/knowledge/decisions/` — source of truth: `api.md`, `domain.md`, `glossary.md`
- `.github/agents/` — custom agents (`ba`, `dev`, `test`)
- `.github/skills/` — detailed best-practice guides, loaded automatically by task context:
  `nextjs-app-router`, `nextjs-server-actions`, `prisma-postgres`, `postgres-performance`,
  `prisma-testing-seeding`, `vercel-deployment`, `api-design`, `code-review`

> This file covers project facts and hard rules only. For implementation detail and best practices, see the skill matching the task at hand.

## Core conventions

- Naming: camelCase variables/functions, PascalCase components, kebab-case route files
- Server Component is the default; add `"use client"` only when needed
- All DB access goes through the Prisma singleton in `lib/db.ts` — never instantiate `PrismaClient` elsewhere
- Validate all external input (API body, Server Action input) with zod before it touches the DB
- API/Server Action errors follow a consistent shape: `{ error: { code, message } }`

## Workflow for DB-related changes

1. Edit `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name <migration-name>`
3. Update `docs/knowledge/decisions/domain.md` if the business model changed
4. Update `docs/knowledge/decisions/api.md` if the API contract changed

## Git workflow

- Branch: `feature/<short-name>`, `fix/<short-name>`
- Commit message: `<type>: <short-description>` (e.g. `feat: add user signup api`)
- Every change gets reviewed before merge — see the `code-review` skill

## Boundaries — DO NOT

- Do not hand-edit files in `prisma/migrations/` that have already been committed
- Do not change the Postgres schema without recording the decision in `docs/knowledge/decisions/`
- Do not hardcode secrets/connection strings — always use environment variables via Vercel Project Settings (local dev uses `.env`, never commit a real `.env`)
- Do not drop or rename DB columns without a migration accompanied by a data-migration plan
- Do not run tests, seeds, or migrations against the production database
