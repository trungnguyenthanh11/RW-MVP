---
name: prisma-testing-seeding
description: Test database isolation, seeding, and fixture conventions for RW-MVP. Use when writing a unit/integration test or Playwright E2E test that touches the database.
---

# Prisma Testing & Seeding — RW-MVP

## Overview

Any test that touches Postgres must run against an isolated test database — never local dev data, and never production. This skill covers seeding, fixtures, and test-data isolation for both unit/integration tests and Playwright E2E.

**Trigger:** writing a test (unit, integration, or Playwright E2E) that reads or writes through Prisma.

## Test database isolation

- Tests run against a dedicated test `DATABASE_URL` (e.g. `.env.test`) — never the same URL as local dev or production. Confirm this before running any test that mutates data.
- Each test sets up its own data and tears it down — no test should depend on data left over from a previous test or a manually-seeded dev database.
- Prefer wrapping each test in a transaction that's rolled back afterward (where the test runner supports it) over manual `deleteMany()` cleanup, for both speed and isolation guarantees.

## Seeding

- Reusable seed/fixture data lives in `prisma/seed.ts` (for local dev bootstrap) or a `test/fixtures/` helper (for test-specific factories) — don't inline large object literals repeatedly across test files.
- Build fixture helpers as functions (`createTestUser(overrides?)`) that return sensible defaults with any field overridable — not fixed JSON blobs that every test has to work around.
- Seed only what a test needs to exercise its scenario — don't seed the entire domain graph for a test that touches one model.

## Unit/integration tests

- Mock Prisma only for true unit tests of business logic that don't need real DB behavior (e.g. a pure function). For anything validating a query's actual behavior (filters, joins, constraints), run against the real (test) Postgres instance — a mocked Prisma client won't catch a broken `@@unique` or a bad `include`.
- Assert on the shape of data returned, not on Prisma being called with certain arguments — that couples the test to implementation, not behavior.

## Playwright E2E

- Set up required test data via a Prisma seed helper or a setup API call in `test.beforeEach` — never rely on the click-through UI to create prerequisite data.
- Tear down created data in `test.afterEach`/`afterAll` so runs don't accumulate state across CI runs.
- Each E2E test must be runnable independently and in any order — no test assumes another test ran first.

## Common mistakes to catch

- A test pointed at the dev or production `DATABASE_URL`.
- Tests that pass only when run in a specific order (shared mutable state).
- Large inline fixture objects duplicated across multiple test files instead of a shared factory.
- A "unit test" that mocks Prisma but is actually asserting on query shape/constraints — misses real DB behavior.
- E2E test with no teardown, leaving orphaned rows in the test database.
