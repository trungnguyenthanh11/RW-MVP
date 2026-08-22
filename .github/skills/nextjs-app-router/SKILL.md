---
name: nextjs-app-router
description: Conventions for routing, layouts, and Server/Client Components in the Next.js App Router for RW-MVP. Use when creating or editing anything under app/ — a page, layout, route handler, or loading/error boundary.
---

# Next.js App Router — RW-MVP

## Overview

`app/` is the only routing surface. Server Components are the default rendering mode; `"use client"` is the exception, not the norm. This skill governs what goes where and when to cross the server/client boundary.

**Trigger:** creating/editing a route (`app/**/page.tsx`), a layout, a loading/error boundary, or deciding whether a component needs `"use client"`.

## Directory conventions

- Route segment = folder; `page.tsx` is the leaf route, `layout.tsx` wraps it and everything below it.
- Route Handlers live at `app/api/**/route.ts` — REST endpoints only, never a place for page markup.
- Colocate a route's private components under `app/**/_components/` (underscore prefix excludes it from routing) instead of leaking one-off components into `components/`.
- `loading.tsx` / `error.tsx` per segment where a route does async data fetching — don't rely on a single global spinner.

## Server vs Client Components

- **Default to Server Component.** Only add `"use client"` when the file needs: state/effects (`useState`/`useEffect`), browser-only APIs, event handlers, or a third-party client-only library.
- Push `"use client"` as far down the tree as possible — wrap just the interactive leaf (e.g. a button, a form), not the whole page. A client boundary pulls everything inside it into client JS.
- Never import server-only code (Prisma, `lib/db.ts`, secrets) into a file marked `"use client"` — it will either fail the build or leak into the browser bundle.
- Pass data down from Server → Client Components as serializable props; don't pass functions, class instances, or Prisma model instances directly — map to plain objects/DTOs first.

## Data fetching

- Fetch data directly in Server Components with `async`/`await` — no `useEffect` + `fetch` pattern for initial page data.
- Use Next.js `fetch` caching (`cache: 'force-cache'` / `'no-store'`, or `revalidate`) deliberately; know which one you're picking, don't leave it to the default silently.
- For DB reads, call `lib/db.ts` (the Prisma singleton) directly from the Server Component or a `lib/` helper — don't round-trip through your own Route Handler from server code.

## Metadata & SEO

- Use the `metadata` export (or `generateMetadata`) per route instead of manually injecting `<head>` tags.

## Common mistakes to catch

- A Server Component importing a hook (`useState`, `useRouter` from `next/navigation` in a client-only way) without `"use client"`.
- Prisma/`lib/db.ts` imported into a `"use client"` file.
- A whole page marked `"use client"` because one button needs an `onClick`.
- Route Handler (`app/api/**/route.ts`) used to serve page markup instead of JSON.
- Missing `loading.tsx`/`error.tsx` on a route that awaits a slow DB call.
