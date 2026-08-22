---
name: postgres-performance
description: Query and index performance checklist for PostgreSQL/Prisma in RW-MVP. Use when writing or reviewing any query that lists, filters, sorts, or aggregates data, or when a query is suspected to be slow.
---

# Postgres Performance — RW-MVP

## Overview

Correctness first, but a query that's O(n) round-trips or missing an index will not survive real data volume. This skill is a targeted checklist, not general DB theory — use it while writing or reviewing list/filter/aggregate queries.

**Trigger:** writing a query that lists, filters, sorts, paginates, or aggregates; or reviewing a slice for performance.

## N+1 detection

- Any loop (`for`, `.map`, `.forEach`) that calls a Prisma query per iteration is an N+1 — replace it with one query using `include`/`select` with a `where: { id: { in: [...] } }`, or a single query with nested relations.
- Watch for this pattern hiding inside a React Server Component that maps over a list and fetches per item — the fix is the same: fetch once with the relation included.

## Indexing

- Every column used in a `WHERE`, `ORDER BY`, or `JOIN` (via a Prisma relation) needs an index — check `@@index`/`@unique` exists in `schema.prisma` before assuming a query will scale.
- Composite filters (e.g. `WHERE status = ? AND projectId = ?`) want a composite index matching the filter order, not two separate single-column indexes.
- Don't over-index — every index costs write performance; only add one backed by an actual query pattern.

## Pagination

- Any endpoint/query returning a list must paginate (`take`/`skip` or cursor-based with `cursor`/`take`) — never return an unbounded `findMany()` on a table that grows with usage.
- Prefer cursor-based pagination (stable under concurrent inserts) over offset (`skip`) for anything beyond a small admin list.

## Selecting only what's needed

- Use `select` to fetch only the fields a page/component actually renders — don't default to fetching the whole row when 3 fields are used.
- Use `_count` instead of `include`-ing a full relation just to display a number.

## Connection pooling (serverless)

- Vercel serverless functions don't share a long-lived process — each cold start can open a new Postgres connection. Confirm the deployed `DATABASE_URL` goes through a pooler (e.g. PgBouncer/Prisma Accelerate) rather than a direct connection, per the `vercel-deployment` skill.
- Never assume two requests share the same Prisma connection/instance.

## Common mistakes to catch

- Query-per-loop-iteration (N+1).
- `findMany()` with no `take`/pagination on a growing table.
- Filtered/sorted column with no matching index.
- Full relation `include`-d just to show a count.
- Direct (non-pooled) Postgres connection string used for the deployed app.
