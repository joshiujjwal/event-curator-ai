# AGENTS.md — EventCuratorAI

Instructions for AI coding agents (Codex, Copilot Workspace, etc.).

---

## Setup

```bash
# 1. Install dependencies
npm install

# 2. Configure environment
cp .env.example .env.local
# Set DATABASE_URL, OPENAI_API_KEY, NEXTAUTH_SECRET, OAuth keys

# 3. Start PostgreSQL (with pgvector)
# Requires PostgreSQL 15+ and pgvector extension

# 4. Run migrations
npx prisma migrate dev

# 5. Verify setup
npm run test        # Should pass smoke test
npm run typecheck   # Should exit 0
```

---

## Code Style

### TypeScript
- **Strict mode is non-negotiable** — `strict: true` in tsconfig. No `any`.
- Use `type` for unions/intersections, `interface` for object shapes that may be extended
- Prefer `const` over `let`. Never use `var`.
- Use `satisfies` for object literals that should be type-checked: `const config = { ... } satisfies Config`
- Import order: Node built-ins → external packages → internal (`@/`) → relative

### Next.js App Router
- Page components are in `src/app/<route>/page.tsx` — they are React Server Components by default
- Add `'use client'` only when you need hooks, event handlers, or browser APIs
- API routes live in `src/app/api/<path>/route.ts` — export named functions `GET`, `POST`, etc.
- Keep route handlers thin: validate input → call `src/lib/` function → return Response

### React Components
- One component per file in `src/components/<ComponentName>.tsx`
- Props interfaces named `<ComponentName>Props`
- No data fetching inside components — use hooks from `src/hooks/` or pass data as props

### Source Adapters
- Implement the `SourceAdapter` interface from `src/lib/sources/base.ts`
- File name: `src/lib/sources/<sourcename>.ts` (lowercase, no hyphens)
- Class name: `<Sourcename>Adapter` (PascalCase)
- Never throw raw errors — wrap in a typed `IngestionError` with `source` and `originalError` fields

---

## Testing (Red/Green TDD)

**Write the test before the implementation. Every time.**

```bash
npm run test          # Run all unit + integration tests
npm run test:watch    # Watch mode (use during development)
npm run test:e2e      # Playwright E2E (requires dev server on :3000)
```

### Test File Locations
- Unit tests: `tests/unit/<mirrors src/ path>.test.ts`
- Integration tests: `tests/integration/<feature>.test.ts`
- E2E tests: `tests/e2e/<flow>.spec.ts`

### Test Rules
- **No live API calls in unit tests** — always mock external HTTP with `vi.mock` or `msw`
- **No live OpenAI calls in unit tests** — mock the `openai` client
- Integration tests use a real test database — set `DATABASE_URL` in `.env.test`
- Each test file imports from `@/lib/...` (not relative paths)
- Tests must be deterministic — no `Date.now()` or `Math.random()` without mocking

### What to Test
- Source adapters: field mapping, pagination, error handling, missing optional fields
- AI functions: correct prompts sent to OpenAI (mocked), correct return types
- API routes: correct status codes, correct response shapes, auth rejection
- DB helpers: upsert idempotency, transaction rollback on error

---

## PR Instructions

Every PR must include:

1. **Evidence of testing**: `npm run test` output (paste the summary line)
2. **TypeScript clean**: `npm run typecheck` exits 0
3. **Lint clean**: `npm run lint` exits 0
4. **Manual test evidence**: screenshot or `curl` command + response for any new API route
5. **TODO.md updated**: check off completed items, add to Lessons Learned if applicable

### PR Title Format
```
feat: <what it does>           # new feature
fix: <what it fixes>           # bug fix
test: <what is tested>         # tests only
refactor: <what changed>       # no behavior change
chore: <tooling/deps>          # non-code changes
```

### Do NOT
- Merge without all tests passing
- Leave TODO comments in committed code (add to TODO.md instead)
- Add `any` types to work around TypeScript errors — fix the type
- Remove existing tests — if a test is wrong, fix the test AND the code together
- Commit `.env.local` or any file containing secrets
