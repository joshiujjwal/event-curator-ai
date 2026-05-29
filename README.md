# EventCuratorAI

> 🚧 **Status: Early Development**

AI-powered platform that aggregates IRL events from multiple sources (Eventbrite, Meetup, Facebook Events, local venues, etc.), personalizes recommendations based on user interests, and provides a unified discovery experience for local events.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS |
| Database | PostgreSQL + Prisma ORM |
| AI / Embeddings | OpenAI API (GPT-4o + text-embedding-3-small) |
| Vector Store | pgvector |
| Auth | NextAuth.js |
| Job Queue | BullMQ + Redis |
| Testing | Vitest + Playwright |
| Deployment | Vercel (frontend) + Railway (services) |

---

## Getting Started

```bash
# Clone
git clone https://github.com/joshiujjwal/event-curator-ai.git
cd event-curator-ai

# Install dependencies
npm install

# Set up environment
cp .env.example .env.local
# Fill in API keys (see docs/spec.md for required vars)

# Run database migrations
npx prisma migrate dev

# Start development server
npm run dev
```

### Available Scripts

```bash
npm run dev          # Start Next.js dev server (localhost:3000)
npm run build        # Production build
npm run test         # Run Vitest unit/integration tests
npm run test:e2e     # Run Playwright end-to-end tests
npm run lint         # ESLint
npm run typecheck    # tsc --noEmit
npm run db:migrate   # Run Prisma migrations
npm run db:seed      # Seed database with sample events
npm run ingest       # Run event ingestion pipeline (all sources)
```

---

## Project Structure

```
event-curator-ai/
├── src/
│   ├── app/                    # Next.js App Router pages & API routes
│   │   ├── api/                # API route handlers
│   │   ├── (auth)/             # Auth pages (login, signup)
│   │   ├── discover/           # Event discovery feed
│   │   ├── event/[id]/         # Event detail page
│   │   └── profile/            # User preference management
│   ├── components/             # React components
│   ├── lib/
│   │   ├── sources/            # Per-source ingestion adapters
│   │   │   ├── eventbrite.ts
│   │   │   ├── meetup.ts
│   │   │   ├── facebook.ts
│   │   │   └── base.ts         # Abstract SourceAdapter interface
│   │   ├── ai/                 # Embedding, ranking, summarization
│   │   │   ├── embeddings.ts
│   │   │   ├── ranker.ts
│   │   │   └── summarizer.ts
│   │   └── db/                 # Prisma client + query helpers
│   ├── hooks/                  # Custom React hooks
│   └── types/                  # Shared TypeScript types
├── tests/
│   ├── unit/                   # Vitest unit tests (mirrors src/)
│   ├── integration/            # API + DB integration tests
│   └── e2e/                    # Playwright browser tests
├── docs/
│   ├── spec.md                 # Feature specification
│   └── adr/                    # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   └── instructions/
├── prisma/
│   └── schema.prisma
├── README.md
├── TODO.md
├── CLAUDE.md
└── AGENTS.md
```

---

## Contributing

1. **Read** `TODO.md` to find the next task
2. **Write failing tests first** (red phase) — no implementation without a test
3. **Implement** until tests pass (green phase)
4. **Review your own diff** before committing — never ship unreviewed code
5. **Commit** with a descriptive message referencing the TODO item
6. **Update** `CLAUDE.md` / `AGENTS.md` with anything you learned

PRs require:
- All tests passing (`npm run test && npm run test:e2e`)
- TypeScript clean (`npm run typecheck`)
- Evidence of manual testing in the PR description (screenshot or curl output)
- Lint clean (`npm run lint`)
