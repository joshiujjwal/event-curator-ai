# EventCuratorAI — Task Breakdown

## How to Use This File

Work one item at a time. Gate each phase behind evidence before proceeding.

**Per-task workflow:**
1. Write failing tests FIRST (red) — commit as `test: <description>`
2. Implement until tests pass (green) — commit as `feat: <description>`
3. Review your own diff manually — no surprises in CI
4. Update `CLAUDE.md` / `AGENTS.md` if you learned something non-obvious
5. Check off the item and move to the next

**Evidence gates between phases:**
- All tests pass: `npm run test`
- TypeScript clean: `npm run typecheck`
- Lint clean: `npm run lint`
- Manual smoke test documented (curl / screenshot in commit message or PR)

---

## Phase 0: Foundation ⬜

> Goal: Zero-friction dev environment. Any engineer can clone → run in < 5 minutes.

- [ ] Init Next.js 14 with TypeScript strict mode: `npx create-next-app@latest . --typescript --tailwind --eslint --app`
- [ ] Configure `tsconfig.json` with `strict: true`, `baseUrl: "."`, path aliases (`@/` → `src/`)
- [ ] Install and configure Vitest + `@testing-library/react` for unit tests
- [ ] Install and configure Playwright for E2E tests
- [ ] Add `npm run typecheck` script (`tsc --noEmit`)
- [ ] Set up Prisma: `npx prisma init` with PostgreSQL provider
- [ ] Create `.env.example` with all required env var keys (no values)
- [ ] Add GitHub Actions CI: on push → lint + typecheck + test
- [ ] Write first smoke test: `tests/unit/smoke.test.ts` — asserts `true === true` (validates test runner works)
- [ ] **GATE**: CI green, `npm run test` passes locally

---

## Phase 1: Data Model & Ingestion Core ⬜

> Goal: Define the event schema and get one source ingesting real data.

- [ ] Write Prisma schema: `Event`, `Source`, `User`, `UserInterest`, `UserEventInteraction` models (see `docs/spec.md` for fields)
- [ ] Write unit tests for Prisma model constraints (required fields, unique indexes)
- [ ] Run `npx prisma migrate dev --name init`
- [ ] Define `SourceAdapter` abstract interface in `src/lib/sources/base.ts` with `fetchEvents(): Promise<RawEvent[]>`
- [ ] Write unit tests for `SourceAdapter` contract (mock implementation)
- [ ] Implement `EventbriteAdapter` in `src/lib/sources/eventbrite.ts`
  - [ ] Write tests with mocked HTTP responses (no live API calls in tests)
  - [ ] Map Eventbrite fields → internal `Event` schema
  - [ ] Handle pagination, rate limits, missing fields
- [ ] Implement `MeetupAdapter` in `src/lib/sources/meetup.ts`
  - [ ] Write tests with mocked HTTP responses
  - [ ] Map Meetup GraphQL response → internal `Event` schema
- [ ] Create `src/lib/ingest.ts` — orchestrator that runs all adapters and upserts to DB
- [ ] Write integration test for ingest pipeline using a local test database
- [ ] **GATE**: `npm run ingest` successfully pulls and persists 10+ events from Eventbrite into local DB

---

## Phase 2: AI Ranking & Personalization ⬜

> Goal: Events are scored and ranked by relevance to a user's stated interests.

- [ ] Write tests for `src/lib/ai/embeddings.ts` (mock OpenAI client)
  - [ ] `embedEvent(event: Event): Promise<number[]>` — embeds title + description
  - [ ] `embedInterests(interests: string[]): Promise<number[]>` — embeds user interest tags
- [ ] Implement `embeddings.ts` using `openai` SDK
- [ ] Add `pgvector` extension to Prisma schema; add `embedding` field to `Event`
- [ ] Write migration to add `vector(1536)` column to events table
- [ ] Write tests for `src/lib/ai/ranker.ts`
  - [ ] `rankEvents(userId, events): Promise<RankedEvent[]>` — cosine similarity against user embedding
- [ ] Implement `ranker.ts`
- [ ] Write tests for `src/lib/ai/summarizer.ts`
  - [ ] `summarizeEvent(event: Event): Promise<string>` — 2-sentence AI summary
- [ ] Implement `summarizer.ts` (GPT-4o-mini, cache in DB)
- [ ] Wire embeddings into ingest pipeline — embed each event after upsert
- [ ] **GATE**: `npm run test` passes, manual test shows events ranked differently for two users with different interests

---

## Phase 3: API Routes & Auth ⬜

> Goal: Authenticated REST/tRPC API that powers the frontend.

- [ ] Install and configure NextAuth.js with Google + GitHub OAuth providers
- [ ] Write tests for auth middleware (`src/app/api/auth/[...nextauth]/route.ts`)
- [ ] Create API routes:
  - [ ] `GET /api/events` — paginated event feed, ranked for authenticated user
  - [ ] `GET /api/events/[id]` — single event detail
  - [ ] `POST /api/events/[id]/save` — save event to user's list
  - [ ] `GET /api/user/interests` — get user interest tags
  - [ ] `PUT /api/user/interests` — update interest tags (re-triggers re-ranking)
- [ ] Write integration tests for each route using `fetch` against a test server
- [ ] Add rate limiting middleware (upstash/ratelimit or simple in-memory for dev)
- [ ] **GATE**: All API routes return correct shapes, auth rejects unauthenticated requests

---

## Phase 4: Frontend ⬜

> Goal: A working discovery UI that users can actually use.

- [ ] Build `EventCard` component with tests (title, date, location, source badge, save button)
- [ ] Build `EventFeed` component — infinite scroll, ranked list
- [ ] Build `EventDetailPage` (`/event/[id]`) — full event info + AI summary
- [ ] Build `InterestSelector` component — tag picker that triggers re-ranking
- [ ] Build `ProfilePage` (`/profile`) — manage interests, saved events
- [ ] Build `DiscoverPage` (`/discover`) — main entry point, search + filter + ranked feed
- [ ] Add source filter (Eventbrite / Meetup / Facebook / All)
- [ ] Add date range filter
- [ ] Add distance filter (requires geolocation)
- [ ] Write Playwright E2E tests:
  - [ ] User can log in and see ranked event feed
  - [ ] User can update interests and feed re-ranks
  - [ ] User can save/unsave events
- [ ] **GATE**: All E2E tests pass, Lighthouse score > 80 on /discover

---

## Phase 5: Additional Sources ⬜

- [ ] `FacebookAdapter` — Facebook Events via Graph API (requires user token)
- [ ] `ICSAdapter` — Parse `.ics` calendar files from local venue websites
- [ ] `TicketmasterAdapter` — Ticketmaster Discovery API
- [ ] Add source health dashboard: last successful sync, event count per source
- [ ] **GATE**: 3+ sources ingesting real events concurrently without conflicts

---

## Phase 6: Background Jobs & Sync ⬜

- [ ] Install BullMQ + Redis
- [ ] Create `ingest` job: runs all adapters on a schedule (every 4 hours)
- [ ] Create `embed` job: processes events without embeddings
- [ ] Create `summarize` job: generates summaries for new events
- [ ] Write tests for job handlers (mock queue, assert DB state after processing)
- [ ] Add admin route to manually trigger sync: `POST /api/admin/sync`
- [ ] **GATE**: Jobs run reliably for 24 hours without memory leaks or failures

---

## Phase 7: Ship ⬜

- [ ] Set up Vercel project, configure env vars
- [ ] Set up Railway for PostgreSQL + Redis
- [ ] Configure custom domain
- [ ] Add error monitoring (Sentry)
- [ ] Add basic analytics (Plausible or Vercel Analytics — privacy-first)
- [ ] Write `docs/runbook.md` — how to deploy, rollback, debug prod
- [ ] Load test: simulate 50 concurrent users browsing `/discover`
- [ ] **GATE**: App handles load test, error rate < 1%, p95 latency < 800ms

---

## Parking Lot 🅿️

> Ideas that aren't in scope yet — revisit after Phase 4

- Push notifications for saved event reminders
- "Bring a friend" sharing flow
- Venue partnership / promoted events
- Calendar sync (Google Calendar export)
- Mobile app (React Native / Expo)
- Social graph — see what friends are attending
- Event clustering by neighborhood / vibe

---

## Lessons Learned 📝

> Add non-obvious discoveries here as you work. This is the compound engineering loop.

<!-- Example:
- Meetup's GraphQL API rate-limits at 10 req/min per IP — use exponential backoff
- pgvector cosine similarity requires vectors to be normalized first for correct results
-->
