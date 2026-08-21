# Tech Stack Rules — RW-MVP

> Fill in during [Part 03 — Harness Setup](../flexible-mob-workshop/workshop/03-harness-setup.md). Reference the existing instruction files rather than duplicating rules.

## Frontend

- **Framework:** Next.js 14+ (App Router), TypeScript
- **Styling:** Tailwind CSS
- **Conventions:** `.github/copilot-instructions.md` — Server Components by default; `"use client"` only when needed; PascalCase components, kebab-case route files

## Backend

- **Framework:** Next.js Route Handlers (`app/api/**/route.ts`) + Server Actions (`lib/actions/`)
- **ORM:** Prisma — singleton at `lib/db.ts`, never instantiate `PrismaClient` elsewhere
- **Validation:** zod on all external input before it touches the DB
- **Error shape:** `{ error: { code, message } }` at all API/Server Action boundaries

## Data / storage

- **Database:** PostgreSQL (local via `.env`, production via Vercel environment variables)
- **Migrations:** `npx prisma migrate dev --name <name>` locally; record every schema change in `docs/knowledge/decisions/domain.md`
- **Never** hand-edit committed migration files in `prisma/migrations/`

## Build & run commands

| Action | Command |
|---|---|
| Install | `npm install` |
| Run dev | `npm run dev` |
| Build | `npm run build` |
| Test | `npm test` |
| E2E | `npx playwright test` |
| Lint | `npm run lint` |
| DB migrate | `npx prisma migrate dev --name <name>` |
| DB studio | `npx prisma studio` |

## AI-feature specifics

- Provider / model: set per feature (e.g. OpenAI, Anthropic) — never commit API keys
- API key location: `.env` locally → Vercel Project Settings in production
- Fallback: return a graceful error response if the API call fails or is rate-limited

## Key constraints (from `.github/copilot-instructions.md`)

- Do not hardcode secrets — always use environment variables
- Do not drop or rename DB columns without a migration + data-migration plan
- Do not run tests, seeds, or migrations against the production database
