---
name: code-review
description: Conducts multi-axis code review for RW-MVP (Next.js + Prisma/PostgreSQL + Vercel). Use before merging any change, after completing a feature implementation, or when reviewing code written by the dev-agent or a human.
---

# Code Review — RW-MVP

## Overview

Every change gets reviewed before merge — no exceptions. Review covers five axes: correctness, readability, architecture, security, and performance, plus project-specific checks for this stack.

**The approval standard:** Approve a change when it definitely improves overall code health, even if it isn't perfect. Don't block a change because it isn't exactly how you would have written it — if it follows the project's conventions (see `copilot-instructions.md` and the relevant skills) and improves the codebase, approve it.

## The Five-Axis Review

### 1. Correctness

- Does it match the BA spec / acceptance criteria?
- Are edge cases handled (null, empty, boundary values, missing relations)?
- Are error paths handled, not just the happy path (see `api-design` skill error shape)?
- Do tests exist and actually test behavior, not implementation details?
- Any off-by-one, race conditions, or state inconsistencies — especially around concurrent writes to the same Postgres rows?

### 2. Readability & Simplicity

- Names descriptive, consistent with project convention (camelCase vars, PascalCase components)
- Straightforward control flow — no deeply nested ternaries or callback pyramids
- Could this be done in fewer lines? Are abstractions earning their complexity (don't generalize until the third use case)?
- Any dead code, unused imports, or leftover `console.log`/commented-out blocks?
- Is a new conditional bolted onto an unrelated flow instead of its own helper?

### 3. Architecture

- Follows existing patterns in `app/`, `lib/`, `prisma/` — no new pattern introduced without justification
- No duplicated logic that should live in a shared `lib/` helper
- Server Component vs `"use client"` boundary respected (see `nextjs-app-router` skill) — no unnecessary client boundary pulling server-only code with it
- Does this refactor reduce complexity or just relocate it?
- Are type boundaries explicit — question gratuitous `any`, unchecked casts, or silent fallbacks around Prisma results

### 4. Security

- User input validated with zod at every boundary (API route, Server Action) before touching Prisma
- No raw string concatenation into `$queryRaw` — always parameterized
- Secrets/connection strings never hardcoded — always via env vars (see `vercel-deployment` skill)
- Auth/authorization checked on every mutating route/action, not just the UI hiding a button
- External data (webhooks, third-party API responses) treated as untrusted before use

### 5. Performance

- Any N+1 query pattern — looping and querying per item instead of one `include`/`select` (see `postgres-performance` skill)
- Any missing index on a new foreign key or frequently filtered column
- Any missing pagination on a list endpoint/query
- Any large relation fetched via `include` when only a count or preview was needed (`_count` instead)
- Any Server Action mutation missing `revalidatePath`/`revalidateTag` (see `nextjs-server-actions` skill) — stale UI isn't a performance bug per se, but treat it with the same rigor here

## Project-Specific Checklist (RW-MVP)

- [ ] All DB access goes through the Prisma singleton (`lib/db.ts`) — no `new PrismaClient()` elsewhere
- [ ] Any `prisma/schema.prisma` change has a matching migration committed (`prisma/migrations/`), and no existing migration was hand-edited
- [ ] Any API/domain contract change is reflected in `docs/knowledge/decisions/api.md` / `domain.md` in the same change
- [ ] Route Handlers/Server Actions default to Node.js runtime unless Edge compatibility with Prisma was explicitly verified
- [ ] New/changed queries checked against the connection-pooling setup — no assumption that Vercel serverless functions share a connection
- [ ] Test coverage exists: unit/integration for logic and API, Playwright E2E for new user-facing flows (see `prisma-testing-seeding` skill for test DB isolation)
- [ ] No test or seed script points at the production `DATABASE_URL`

## Categorizing Findings

Label every comment with severity so the author knows what's required vs optional:

| Prefix                        | Meaning            | Author Action                                                           |
| ----------------------------- | ------------------ | ----------------------------------------------------------------------- |
| _(no prefix)_                 | Required change    | Must address before merge                                               |
| **Critical:**                 | Blocks merge       | Security vulnerability, data loss, broken migration, missing auth check |
| **Nit:**                      | Minor, optional    | Formatting, naming preference                                           |
| **Optional:** / **Consider:** | Suggestion         | Worth considering, not required                                         |
| **FYI**                       | Informational only | No action needed                                                        |

**Lead with what matters.** Order findings by leverage: correctness/security/migration-safety first, then structural issues, then nits. A few high-conviction comments beat a long list.

## Review Process

1. **Understand context** — what BA spec/task does this implement? What behavior should change?
2. **Review tests first** — do they exist, cover edge cases, and would they catch a regression?
3. **Walk the five axes** for each changed file
4. **Verify the verification** — did the author run lint/build/tests? Any manual check for a migration or Server Action revalidation?

## Change Sizing

```
~100 lines changed   → Good, reviewable in one sitting.
~300 lines changed   → Acceptable if it's a single logical change.
~1000 lines changed  → Too large, split it (e.g. schema change in one PR, consuming feature in the next).
```

Separate refactoring from feature work — a change that both refactors and adds behavior is two changes.

## Dead Code Hygiene

After any refactor or feature change, list any now-unused code explicitly and ask before deleting:

```
DEAD CODE IDENTIFIED:
- getLegacyTaskStatus() in lib/tasks.ts — replaced by getTaskStatus()
- OldTaskCard component — replaced by TaskCard
→ Safe to remove these?
```

## Handling Disagreements

1. Technical facts/data override opinions
2. Project conventions (`copilot-instructions.md`, skills) are the authority on style/pattern questions
3. Don't accept "I'll clean it up later" for migration hygiene or missing tests — require it before merge unless it's a genuine emergency, in which case file a follow-up with an owner

## Honesty in Review

- Don't rubber-stamp — "LGTM" without evidence of review helps no one
- Quantify problems when possible: "this N+1 adds a query per task, ~N extra round-trips" beats "this could be slow"
- Push back on approaches with clear problems and propose the alternative directly

## Dependency Discipline

Before adding any dependency: does the existing stack (Next.js/Prisma/zod/Tailwind) already solve this? Check size, maintenance status, and license. Upgrade existing dependencies one at a time, reading the changelog rather than trusting semver, and let the test suite (not "it installed") decide if the upgrade is safe.

## The Review Checklist

```markdown
## Review: [PR/Change title]

### Context

- [ ] I understand what this change implements and why

### Correctness

- [ ] Matches BA spec/acceptance criteria
- [ ] Edge cases and error paths handled
- [ ] Tests cover the change adequately

### Readability

- [ ] Names clear, logic straightforward, no dead code

### Architecture

- [ ] Follows existing app/lib/prisma patterns
- [ ] Server/Client Component boundary correct
- [ ] Refactors reduce complexity rather than relocate it

### Security

- [ ] Input validated with zod at every boundary
- [ ] No hardcoded secrets/connection strings
- [ ] Auth checks in place on mutating routes/actions

### Performance

- [ ] No N+1 patterns; indexes present for new filtered/FK columns
- [ ] Pagination on list endpoints
- [ ] Server Action mutations call revalidatePath/revalidateTag

### Project-specific

- [ ] Prisma singleton used; migration committed and not hand-edited
- [ ] docs/knowledge/decisions/ updated if contract/domain changed
- [ ] Test DB isolated from production

### Verification

- [ ] Tests pass, build succeeds
- [ ] Migration applied cleanly on a fresh DB (if schema changed)

### Verdict

- [ ] **Approve** — Ready to merge
- [ ] **Request changes** — Issues must be addressed
```

## Common Rationalizations

| Rationalization                      | Reality                                                                                             |
| ------------------------------------ | --------------------------------------------------------------------------------------------------- |
| "It works, that's good enough"       | Working code that's unreadable, insecure, or missing a migration/index creates debt that compounds. |
| "We'll clean it up later"            | Later never comes — require cleanup before merge.                                                   |
| "AI-generated code is probably fine" | Dev-agent code needs the same scrutiny as human code — confident and plausible even when wrong.     |
| "The tests pass, so it's good"       | Tests don't catch missing indexes, missing revalidation, or architecture drift.                     |
| "It's just a small schema tweak"     | Small schema diffs still need a reviewed migration and updated `domain.md`.                         |

## Red Flags

- PRs merged without review
- Migration file hand-edited after being committed
- Server Action mutation with no `revalidatePath`/`revalidateTag`
- New foreign key column with no index
- Test or seed script pointed at production `DATABASE_URL`
- "LGTM" with no evidence of actual review
- Large PR that's "too big to review properly" — split it
