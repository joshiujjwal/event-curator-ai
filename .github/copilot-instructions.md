# GitHub Copilot Instructions — EventCuratorAI

## Project Overview

EventCuratorAI is a Next.js 14 + TypeScript application that aggregates IRL events from multiple sources (Eventbrite, Meetup, Facebook Events, Ticketmaster), personalizes a ranked feed using OpenAI embeddings + pgvector cosine similarity, and provides AI-generated event summaries.

---

## Stack

- **Next.js 14** with App Router (React Server Components by default)
- **TypeScript** strict mode — no `any` allowed
- **Tailwind CSS** for styling
- **Prisma** ORM with **PostgreSQL 15+** and **pgvector** extension
- **OpenAI SDK** (`openai` npm package) — text-embedding-3-small + GPT-4o-mini
- **NextAuth.js** (v5 / Auth.js) for Google + GitHub OAuth
- **BullMQ + Redis** for background job queue
- **Vitest** for unit/integration tests
- **Playwright** for E2E tests

---

## Coding Conventions

### TypeScript
- Always use strict types — no `any`, no non-null assertions (`!`) in `src/lib/`
- Use `interface` for extendable shapes, `type` for unions and aliases
- Prisma-generated types come from `@prisma/client` — import them directly
- Path alias `@/` maps to `src/` — always prefer over relative imports for depth > 1

### Next.js App Router
- RSC (React Server Components) by default — add `'use client'` only for interactivity
- API route handlers: export named `GET`, `POST`, etc. — keep them thin, call `src/lib/` for logic
- Use `getServerSession()` from `next-auth` for server-side auth checks

### Source Adapters
- All adapters implement `SourceAdapter` interface from `src/lib/sources/base.ts`
- Upsert events using `@@unique([sourceId, sourceName])` — never insert duplicates
- Map all fields to the internal `RawEvent` type before returning

### AI Integration
- Use `prisma.$queryRaw` for pgvector operations (Prisma doesn't support vector natively)
- Normalize embeddings before cosine similarity
- Always check `aiSummary` field before calling GPT — use cached value if present

---

## Testing Conventions

- **Write tests before implementation** (red/green TDD)
- Unit tests in `tests/unit/` — mirror `src/` structure
- Integration tests in `tests/integration/` — use real test database
- E2E in `tests/e2e/` — Playwright, requires dev server on port 3000
- **Mock all external HTTP** in unit tests (no live Eventbrite/Meetup/OpenAI calls)
- Run `npm run test && npm run typecheck` before every commit

---

## Boundaries

- **Do NOT** refactor files unless asked — scope changes tightly
- **Do NOT** remove existing tests — fix them if they're wrong
- **Do NOT** add `any` types to suppress errors — fix the root cause
- **Do NOT** move files between directories without updating all imports
- **Do NOT** commit `.env.local` or hardcode API keys
- **Do NOT** use `prisma.event.deleteMany()` — events should be archived, not deleted
- **Do NOT** make direct database calls from React components — go through API routes or Server Actions
- When adding a new source adapter, always add corresponding unit tests in `tests/unit/sources/`
