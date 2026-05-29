# CLAUDE.md — EventCuratorAI

Context for AI coding agents. Keep this under 200 lines. Update it when you learn something non-obvious.

---

## Essential Commands

```bash
# Development
npm run dev              # Next.js dev server → http://localhost:3000
npm run build            # Production build (run before deploying)

# Testing — ALWAYS run before committing
npm run test             # Vitest unit + integration tests
npm run test:watch       # Watch mode during TDD
npm run test:e2e         # Playwright E2E (requires dev server running)

# Quality
npm run typecheck        # tsc --noEmit (zero tolerance for type errors)
npm run lint             # ESLint

# Database
npm run db:migrate       # npx prisma migrate dev
npm run db:generate      # npx prisma generate (after schema changes)
npm run db:studio        # Prisma Studio GUI
npm run db:seed          # Seed with sample events

# Ingestion
npm run ingest           # Run all source adapters once (dev/manual trigger)
```

---

## Directory Map

```
src/
├── app/                 # Next.js App Router — pages and API routes ONLY
│   ├── api/             # Route handlers (no business logic here — call lib/)
│   └── (routes)/        # Page components
├── components/          # Reusable React components (no data fetching inside)
├── lib/
│   ├── sources/         # SourceAdapter implementations — one file per source
│   ├── ai/              # OpenAI calls (embeddings, ranker, summarizer)
│   └── db/              # Prisma client singleton + query helpers
├── hooks/               # React hooks (data fetching, state)
└── types/               # Shared TypeScript types (no Prisma types — use generated)
```

---

## Workflow for Every Task

1. **Run tests first**: `npm run test` — confirm baseline before touching anything
2. **Write failing test**: add test in `tests/unit/` or `tests/integration/` — see it fail
3. **Implement**: write only enough code to make the test pass
4. **Typecheck**: `npm run typecheck` — fix any errors before moving on
5. **Lint**: `npm run lint` — fix before committing
6. **Commit**: descriptive message, reference TODO.md item
7. **Update this file** if you discovered something non-obvious

---

## Non-Obvious Conventions

### Source Adapters
- Every adapter goes in `src/lib/sources/<name>.ts` and exports a class implementing `SourceAdapter`
- **Never** make live HTTP calls in tests — mock with `vi.mock` or `msw`
- Adapters must be **idempotent** — running twice must not create duplicates (upsert on `sourceId + sourceName`)
- Always handle: missing optional fields, empty results, API errors, pagination

### AI / Embeddings
- The Prisma client doesn't support pgvector natively — use raw SQL for vector operations: `prisma.$queryRaw`
- Embeddings are 1536-dimensional (text-embedding-3-small). Normalize before cosine similarity.
- GPT-4o-mini for summaries, not GPT-4o — latency and cost matter at scale
- Cache summaries in `Event.aiSummary` — only regenerate if `description` changed

### API Routes
- Route handlers in `src/app/api/` must be thin — import and call functions from `src/lib/`
- Always validate request body with Zod before touching the DB
- Use `getServerSession()` from NextAuth for auth checks — never trust client-side session data

### Database
- Prisma client is a singleton in `src/lib/db/client.ts` — never instantiate `new PrismaClient()` elsewhere
- Use `prisma.$transaction()` for multi-step writes
- Never use `prisma.event.deleteMany()` in production — soft-delete or archive instead

### Testing
- Unit tests: `tests/unit/<mirrors src path>.test.ts`
- Integration tests: require `DATABASE_URL` pointing to a test database (set in `.env.test`)
- E2E tests: require dev server (`npm run dev`) running on port 3000

### TypeScript
- `strict: true` — no `any`, no `!` non-null assertions in `src/lib/`
- Use `satisfies` operator for type-checked object literals
- Prisma-generated types live in `@prisma/client` — import from there, not from `src/types/`

---

## Environment Variables

See `.env.example` for all required keys. Key ones:

```
DATABASE_URL            # PostgreSQL connection string (with pgvector)
OPENAI_API_KEY          # OpenAI — embeddings + summarization
NEXTAUTH_SECRET         # Random 32-char string
NEXTAUTH_URL            # http://localhost:3000 in dev
GOOGLE_CLIENT_ID / GOOGLE_CLIENT_SECRET
GITHUB_CLIENT_ID / GITHUB_CLIENT_SECRET
EVENTBRITE_API_KEY
MEETUP_CLIENT_ID / MEETUP_CLIENT_SECRET
TICKETMASTER_API_KEY
REDIS_URL               # For BullMQ job queue
```

---

## Known Gotchas

<!-- Add here as you discover them -->
- pgvector requires PostgreSQL 15+ and the extension enabled: `CREATE EXTENSION IF NOT EXISTS vector;`
- Meetup's API is GraphQL but uses non-standard pagination (keysetPagination cursor)
- Eventbrite pagination uses `page` (not cursor) — handle `has_more_items` flag
- NextAuth v5 (Auth.js) has a different import path than v4 — confirm version before using docs
