# DECISIONS.md — Architecture Decision Records

This file records **locked decisions** for ContractIQ. Every decision
here is binding. To change one, add a new ADR that supersedes the
old one — never edit a past ADR silently.

**Format**: each ADR states context, options considered, decision,
rationale (concise), consequences (both directions), and change
control. Cite ADRs by ID in code comments, commit messages, and
rule files.

---

## Index

| ID       | Title                                              | Status   |
| -------- | -------------------------------------------------- | -------- |
| ADR-001  | Backend HTTP layer: Fastify + TS raw + awilix      | Accepted |
| ADR-002  | Frontend framework: Next.js Pages Router           | Accepted |
| ADR-003  | Language & strictness: TypeScript 5.x strict       | Accepted |
| ADR-004  | Monorepo: Turborepo + pnpm workspaces              | Accepted |
| ADR-005  | Database: PostgreSQL via Neon                      | Accepted |
| ADR-006  | ORM: Prisma                                        | Accepted |
| ADR-007  | Vector database: Qdrant Cloud                      | Accepted |
| ADR-008  | Object storage: Cloudflare R2                      | Accepted |
| ADR-009  | Cache & queue: Upstash Redis + BullMQ              | Accepted |
| ADR-010  | Authentication: Better Auth (self-hosted)          | Accepted |
| ADR-011  | Multi-tenancy: workspace-scoped rows + RLS pattern | Accepted |
| ADR-012  | LLM provider strategy: abstraction + fallback      | Superseded by ADR-026 |
| ADR-013  | RAG strategy: contextual retrieval + hybrid + rerank | Accepted |
| ADR-014  | Prompt caching + structured output                 | Accepted |
| ADR-015  | UI foundation: Radix Primitives (no shadcn)        | Accepted |
| ADR-016  | Styling: Tailwind CSS + Radix Colors               | Accepted |
| ADR-017  | Typography stack                                   | Accepted |
| ADR-018  | Iconography: Phosphor Icons                        | Accepted |
| ADR-019  | Testing: full pyramid + LLM evals                  | Accepted |
| ADR-020  | CI/CD: GitHub Actions with 15 gates                | Accepted |
| ADR-021  | Commit convention: Conventional + custom types     | Accepted |
| ADR-022  | Branch strategy: trunk-based short-lived           | Accepted |
| ADR-023  | Documentation: Nextra                              | Accepted |
| ADR-024  | Deployment stack: free-tier first                  | Accepted |
| ADR-025  | Product language: bilingual EN + ID                | Accepted |
| ADR-026  | LLM strategy: free-tier first, paid tier opt-in    | Accepted |

---

---

## ADR-001: Backend HTTP layer — Fastify + TS raw + awilix

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Context**: Need HTTP + application-layer framework for a
multi-tenant SaaS with background jobs, streaming responses, and
many integrations, while treating framework-fluency as a pedagogical
goal.

**Options considered**:
1. NestJS — rejected: reduces the systems-level learning we want.
2. Express — rejected: legacy ecosystem, no first-party TS DX.
3. Hono — rejected: edge-first; not the right fit for jobs and long-lived processes.
4. **Fastify + TS raw + awilix — chosen.**

**Decision**: Fastify 5.x + TypeScript 5.x + `awilix` for DI. Plugins
for auth, tenant context, request logging, error handling.
Fastify-type-provider-zod for schema-driven validation.

**Rationale**:
- Fastify: fastest mainstream Node HTTP, native plugin lifecycle.
- awilix: proven DI, explicit lifetime scopes (singleton/scoped/transient).
- No opinionated framework, so every architectural choice is a
  deliberate learning artifact documented in `learning_docs/`.

**Consequences**:
- (+) Deep understanding of DI, module boundaries, plugin patterns.
- (+) Smaller runtime footprint than NestJS.
- (–) More scaffolding to write ourselves.
- (–) Must document our patterns explicitly for future contributors.

**Cannot change without**: New ADR + explicit Dandi approval.

---

## ADR-002: Frontend framework — Next.js Pages Router

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Context**: Need a mature, well-documented React meta-framework
with strong hosting options and SEO for the landing/marketing area.

**Options considered**:
1. Next.js App Router — rejected: newer, more churn, RSC learning cost.
2. **Next.js Pages Router — chosen.**
3. Remix — rejected: smaller ecosystem for this stack.

**Decision**: Next.js 14.x with Pages Router. Server functions
via API routes. `getServerSideProps`/`getStaticProps` as needed.

**Rationale**:
- Explicit request/response cycle, easier to reason about for
  someone learning meta-framework internals.
- Mature ecosystem, full-featured docs, wider third-party support.

**Consequences**:
- (+) Predictable data fetching model.
- (–) Miss out on RSC and streaming server components.
- (–) Long-term drift risk vs. App Router ecosystem.

**Cannot change without**: New ADR.

---

## ADR-003: Language & strictness — TypeScript 5.x strict

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: TypeScript 5.x with `strict: true`. Additional flags:
`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`,
`noFallthroughCasesInSwitch`. No `any`, no `@ts-ignore`, no
`@ts-nocheck`. Runtime validation via zod at every trust boundary.

**Rationale**: Type safety end-to-end catches whole classes of bugs
before runtime and pairs with zod for external data.

**Consequences**:
- (+) Fewer runtime bugs; refactoring safer.
- (–) Higher friction with poorly-typed third-party libraries; may
  require writing our own `.d.ts` shims occasionally.

**Cannot change without**: New ADR.

---

## ADR-004: Monorepo — Turborepo + pnpm workspaces

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Options considered**:
1. Nx — rejected: heavier, opinionated, plugin-ecosystem overhead.
2. **Turborepo + pnpm — chosen.**
3. Polyrepo — rejected: shared types + coordinated releases outweigh
   the isolation benefit.

**Decision**: Single monorepo, root `pnpm-workspace.yaml`,
`turbo.json` for build/test/lint pipelines. Structure:

```
apps/
  web/          # Next.js Pages Router
  api/          # Fastify
  docs/         # Nextra
packages/
  shared/       # zod schemas, types, constants
  ui/           # component library on Radix primitives
  db/           # Prisma schema + client
  llm/          # LLM provider abstraction
  evals/        # LLM evaluation harness
  config/       # shared tsconfig, biome, tailwind configs
tooling/
  scripts/      # dev scripts
```

**Rationale**: Shared types across web/api, single lockfile, one
CI pipeline, incremental caching via Turborepo.

**Consequences**:
- (+) Type-safe RPC-like contracts (shared package).
- (+) One PR touches full stack.
- (–) Cold-start CI must be tuned (Turborepo remote cache later).

**Cannot change without**: New ADR.

---

## ADR-005: Database — PostgreSQL via Neon (free tier)

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: PostgreSQL 16 hosted on **Neon** (serverless, branching,
free tier 512 MB). Connection via connection pooler (`pgbouncer`
transaction mode). Migration tool: Prisma Migrate (see ADR-006).

**Rationale**:
- Free tier is generous for MVP.
- Database branching enables ephemeral preview environments per PR.
- Serverless suspend/resume aligns with hobby workload.

**Consequences**:
- (+) Zero-cost start, easy scale-up path.
- (–) Cold-start latency after suspend; must warm-up in critical
  paths or use Neon's autoscaling on paid tier later.
- (–) Some PG extensions unavailable on Neon free tier — check
  before designing features that need them.

**Cannot change without**: New ADR.

---

## ADR-006: ORM — Prisma

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Options considered**:
1. Drizzle — rejected: less mature migration tooling.
2. Kysely — rejected: no schema-first model authoring.
3. **Prisma — chosen.**

**Decision**: Prisma 5.x. Schema-first authoring in
`packages/db/schema.prisma`. Prisma Migrate for versioned
migrations. Prisma Client generated into `packages/db/generated`.

**Rationale**: Schema-first is a strong pedagogical anchor; Prisma
Studio speeds local debugging; broad community answers to unblock.

**Consequences**:
- (+) Type-safe queries, good DX.
- (–) Prisma runtime is heavier than raw SQL builders.
- (–) Some advanced SQL patterns require raw queries.

**Cannot change without**: New ADR.

---

## ADR-007: Vector database — Qdrant Cloud

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Options considered**:
1. Pinecone — rejected: stronger vendor lock-in.
2. pgvector — rejected: fine for scale but keeps two workloads on
   one DB in a small free tier.
3. Weaviate — rejected: heavier ops.
4. **Qdrant Cloud — chosen.**

**Decision**: Qdrant Cloud free cluster (1 GB) for MVP. Hybrid
search (dense + BM25 sparse) native. Self-host path available if
we outgrow free tier.

**Rationale**: Free tier is real, open-source escape hatch exists,
native hybrid search is a differentiator for our RAG (ADR-013).

**Consequences**:
- (+) Room to grow; migration to self-hosted preserves API surface.
- (–) Latency depends on Qdrant Cloud region; must select the
  region closest to Fly.io backend.

**Cannot change without**: New ADR.

---

## ADR-008: Object storage — Cloudflare R2

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Cloudflare R2 for contract file storage (PDF, DOCX).
Free tier: 10 GB, 1M Class A ops/month, **zero egress fees**.
S3-compatible API via AWS SDK v3.

**Rationale**: Zero egress is a decisive cost lever for a
document-heavy app.

**Consequences**:
- (+) Predictable cost model.
- (–) Fewer regions than S3; latency for uploads outside Cloudflare's
  network may be higher.

**Cannot change without**: New ADR.

---

## ADR-009: Cache & queue — Upstash Redis + BullMQ

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Upstash Redis (free tier, 10k commands/day,
serverless HTTP) as cache + BullMQ backing store. Long-running
jobs (LLM extraction, embedding, re-ranking) run in BullMQ
workers deployed on Fly.io separate process.

**Rationale**: Free tier plus HTTP-native access removes VPC
complexity for early stage.

**Consequences**:
- (+) Zero infra maintenance.
- (–) Free tier command budget is real; must batch reads and use
  cache keys judiciously (see R500 for AI-integration rules).

**Cannot change without**: New ADR.

---

## ADR-010: Authentication — Better Auth

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Better Auth (self-hosted, TypeScript-native). Plugins
enabled: `organization`, `admin`, `two-factor`, `passkey`, `api-key`.
Session storage in Postgres (not JWT stateless) to allow revocation.

**Rationale**: Self-hosted preserves control; org plugin gives
first-class multi-tenant workspace + membership + role model.

**Consequences**:
- (+) No vendor lock-in; source-code visibility.
- (+) Multi-tenant primitives without hand-rolling.
- (–) We own upgrade cadence and any patched CVEs.
- (–) Some flows (impersonation, admin panel) need custom UI on top.

**Cannot change without**: New ADR + threat model review.

---

## ADR-011: Multi-tenancy — workspace-scoped rows + RLS pattern

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Every tenant-scoped table has a mandatory
`workspace_id` column with an index and foreign key to `workspaces`.
Access enforced at two layers:

1. **Application layer**: request context (`AsyncLocalStorage`)
   carries `workspace_id`; every DB query goes through a
   `TenantScopedRepository` wrapper that auto-injects the WHERE
   clause.
2. **Database layer**: Postgres row-level security policies as a
   defense-in-depth backstop.

Vector store: Qdrant collections partitioned by workspace, or
payload filter `workspace_id` — decided per index (see R500).

**Rationale**: Two independent enforcement layers means one bug
does not leak tenants.

**Consequences**:
- (+) Strong isolation guarantees.
- (–) Extra code discipline required (any raw query must set
  `SET LOCAL` or bypass RLS deliberately, logged).

**Cannot change without**: New ADR.

---

## ADR-012: LLM provider strategy — abstraction + fallback

**Status**: Superseded by ADR-026 (2026-07-21) &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: `packages/llm` defines a `LLMProvider` interface
with methods `complete`, `stream`, `embed`, `rerank`. Concrete
adapters:

| Env         | Primary                  | Fallback            |
| ----------- | ------------------------ | ------------------- |
| development | Google Gemini 2.0 Flash  | Groq (Llama 3.3 70B) |
| production  | Anthropic Claude Haiku 4.5 | Google Gemini 2.0 Flash |

Provider selection via env var `LLM_PROVIDER=gemini|claude|groq`,
plus per-task overrides (`RISK_ANALYSIS_PROVIDER=claude`).

Embeddings: use provider's native embedding endpoint where
available; else fall back to open-source `bge-m3` self-hosted.

**Rationale**: Portfolio-relevant AI-native pattern; also lets us
swap models when free-tier budgets change or a new frontier model
appears.

**Consequences**:
- (+) Model-agnostic system; easy A/B on quality vs cost.
- (–) Adapter surface must be conservative (lowest-common-denominator
  features).
- (–) Structured output (ADR-014) has quirks per provider; adapters
  normalize.

**Cannot change without**: New ADR.

---

## ADR-013: RAG strategy — contextual retrieval + hybrid + rerank

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Retrieval pipeline for long contracts and playbook
knowledge base:

1. **Semantic chunking** on section/clause boundaries (not fixed
   token size), with overlap ≥15%.
2. **Contextual retrieval**: prepend a 1–2 sentence context blurb
   to each chunk before embedding (per Anthropic technique).
3. **Hybrid search**: dense (embedding) + sparse (BM25) via
   Qdrant's built-in hybrid.
4. **Re-ranking**: top-K (K=30) → cross-encoder rerank → top-N
   (N=5). Reranker: `bge-reranker-v2-m3` self-hosted or Cohere
   Rerank free tier.

**Rationale**: State-of-the-art accuracy for legal/contract text,
which is heavy on domain vocabulary and cross-references.

**Consequences**:
- (+) High recall + precision at retrieval.
- (–) Pipeline complexity; must instrument each stage with OTel
  spans for debugging (see R500).

**Cannot change without**: New ADR.

---

## ADR-014: Prompt caching + structured output

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**:

- **Prompt caching**: on from day 1. Cache targets: system prompt,
  tool definitions, few-shot examples, and long contract text
  during multi-turn Q&A sessions. Providers without native caching
  (Gemini): use adapter-level cache-key semantics for provider parity
  where possible; document the divergence when not.
- **Structured output**: strict JSON schema (zod-first). Every tool
  return type and every LLM completion has a schema. Non-conforming
  output is rejected → retried with `schema-repair` prompt → hard
  fail with structured error after N=2 retries.

**Rationale**: Caching cuts token cost ~90% on hot paths; strict
schemas make LLM output reliable enough to persist to DB.

**Consequences**:
- (+) Cost efficiency; reliable downstream code.
- (–) Some provider divergence to handle in adapters.

**Cannot change without**: New ADR.

---

## ADR-015: UI foundation — Radix Primitives (no shadcn/ui)

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Radix UI Primitives (unstyled, accessible headless
components) styled with Tailwind directly in
`packages/ui`. No shadcn/ui. Every component is bespoke to
ContractIQ's design language (R600).

**Rationale**: Portfolio impact — clear signal of independent
design taste; avoids the "shadcn look" that reads as AI-generated.

**Consequences**:
- (+) Distinctive UI; full ownership of markup and styling.
- (–) Higher upfront effort per component; must invest in
  `packages/ui` early.

**Cannot change without**: New ADR + design review.

---

## ADR-016: Styling — Tailwind CSS + Radix Colors

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Tailwind CSS 4.x. Design tokens mapped from Radix
Colors 12-step scales into Tailwind theme. Base neutral: `sand`.
Accent: `iris`. Semantic (`red`, `green`, `amber`) from Radix.
Dark mode via `data-theme` attribute, not class.

**Rationale**: Radix Colors are OKLCH-based with WCAG-tested
contrast pairs at each step; Tailwind gives ergonomic authoring.

**Consequences**:
- (+) Consistent contrast across themes; disciplined design system.
- (–) 12-step scales require education for the team on which step
  to use where (documented in R600).

**Cannot change without**: New ADR.

---

## ADR-017: Typography stack

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**:

- **Display / headings**: Instrument Serif (Fontshare, free).
- **Body / UI**: Inter (with `ss01` feature enabled).
- **Data / code / IDs / timestamps**: Geist Mono (Vercel, free).

Self-host all fonts (no Google Fonts CDN in production) via
`@next/font` local pipeline for privacy and performance.

**Rationale**: Serif display gives editorial-legal mood; Inter is
proven; Geist Mono is a distinctive-but-restrained monospace.
Together they mark the UI as human-crafted (R601).

**Consequences**:
- (+) Distinct visual identity; strong hierarchy.
- (–) Three fonts to self-host and subset; discipline required in
  R602 to use each in the right context.

**Cannot change without**: New ADR + design review.

---

## ADR-018: Iconography — Phosphor Icons

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Phosphor Icons, Regular weight as default. Bold for
emphasis states. No Lucide, no Heroicons, no emoji-as-icon (R601).

**Rationale**: Distinctive character vs the ubiquitous Lucide look;
6 weights available for later expressiveness.

**Consequences**:
- (+) Visual differentiation; large library.
- (–) Package size — tree-shake per import path only.

**Cannot change without**: New ADR.

---

## ADR-019: Testing — full pyramid + LLM evals

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**:

- **Unit**: Vitest.
- **Integration**: Vitest + Testcontainers (real Postgres, Redis,
  Qdrant in Docker).
- **E2E**: Playwright.
- **API contract**: schema-driven via zod + `openapi-typescript`.
- **LLM-specific**:
  - Snapshot/replay of real LLM responses (VCR pattern).
  - LLM-as-judge for accuracy and hallucination grading.
  - Golden dataset (50–100 sample contracts + expected outputs);
    regression run on every prompt/tool schema change.
  - Property tests for structured output shape (fast-check + zod).
- **Load**: k6 for HTTP and streaming endpoints.
- **Coverage target**: 80% overall, 90% for `packages/llm` and
  business-critical services.

**Rationale**: Only way to ship AI features with confidence.

**Consequences**:
- (+) Regressions caught before production; safe prompt iteration.
- (–) LLM golden dataset needs curation and periodic refresh.

**Cannot change without**: New ADR.

---

## ADR-020: CI/CD — GitHub Actions, 15 gates

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: GitHub Actions. On PR, all gates must pass before
merge is allowed:

1. Semantic PR title (Conventional Commits format).
2. Biome lint (replaces ESLint + Prettier).
3. TypeScript `tsc --noEmit` per package, Turborepo cached.
4. Unit tests (affected packages).
5. Integration tests (with Testcontainers).
6. Build all apps.
7. Playwright E2E on Vercel preview.
8. LLM eval (golden dataset) — required when PR touches `packages/llm`
   or prompt files.
9. Trivy container scan.
10. Snyk dependency scan.
11. GitLeaks secret scan.
12. Bundle size limit (`size-limit`) for `apps/web`.
13. Lighthouse CI (perf & a11y ≥95).
14. CodeQL SAST.
15. GitHub Dependency Review.

Release: Changesets on merge to `main` → auto version, changelog,
tag, deploy. Renovate for dependency updates.

**Rationale**: Ships portfolio-grade quality without manual gatekeeping.

**Consequences**:
- (+) Verified quality on every merge.
- (–) Long PR cycle time; must optimize with Turborepo caching and
  parallel jobs.

**Cannot change without**: New ADR.

---

## ADR-021: Commit convention — Conventional + custom types

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Conventional Commits with format `type(scope): subject`.

**Types**:

| Type       | Meaning                                            |
| ---------- | -------------------------------------------------- |
| `feat`     | New feature                                        |
| `fix`      | Bug fix                                            |
| `refactor` | Restructure without behavior change                |
| `perf`     | Performance improvement                            |
| `test`     | Add or update tests                                |
| `docs`     | Documentation only                                 |
| `chore`    | Maintenance, dependency bumps                      |
| `ci`       | CI/CD configuration                                |
| `build`    | Build system or bundler                            |
| `ai`       | Prompt, tool schema, or LLM config changes         |
| `db`       | Database schema or migration                       |
| `sec`      | Security-related change                            |

**Scopes**: `api`, `web`, `shared`, `ui`, `db`, `llm`, `evals`,
`docs`, `harness`, `ci`.

Rules: atomic commits, imperative subject ≤72 char, body wrap at
100 col, footer for `BREAKING CHANGE:` or `Refs: #123`. Signed
commits (SSH signing).

Enforcement: `commitlint` via Lefthook `commit-msg` hook.

**Consequences**:
- (+) Machine-readable history; automated changelog.
- (–) Small learning cost for the human; enforced by hook.

**Cannot change without**: New ADR.

---

## ADR-022: Branch strategy — trunk-based, short-lived branches

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**:

- `main` is always deployable.
- Feature branches: `feat/<short-name>`, `fix/<short-name>`,
  `chore/<short-name>`. Max 2 days age.
- PR to `main`, self-review, merge via **squash** (default) or
  **rebase** (when atomic-commit history is already clean).
- No `develop` or long-lived integration branches.

**Rationale**: Solo dev speed; still keeps PR discipline for
portfolio-visible review history.

**Consequences**:
- (+) Fast iteration; clean history.
- (–) No parallel long-lived features; must feature-flag.

**Cannot change without**: New ADR.

---

## ADR-023: Documentation — Nextra

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: Nextra as `apps/docs`. Sections:

- Getting Started
- Architecture (C4 diagrams)
- AI System (prompts, tools, evals)
- API Reference (auto-gen from OpenAPI)
- Data Model (ERD)
- Deployment
- ADRs (mirrored from this file)
- Contributing
- Security

Storybook lives separately at `apps/web-storybook` for UI components.

**Rationale**: Public, portfolio-visible docs are a differentiator.

**Consequences**:
- (+) Strong external signal.
- (–) Docs must be kept current; enforced by CI check that
  `DECISIONS.md` and mirror in docs are in sync.

**Cannot change without**: New ADR.

---

## ADR-024: Deployment stack — free-tier first

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**:

| Layer          | Service         | Notes                          |
| -------------- | --------------- | ------------------------------ |
| Frontend       | Vercel Hobby    | Preview per PR                 |
| Backend API    | Fly.io          | 3 shared VMs free              |
| Docs site      | Vercel Hobby    | Separate project               |
| Storybook      | Chromatic free  | Optional                       |
| Postgres       | Neon            | 512 MB free, branching         |
| Redis          | Upstash         | 10k cmd/day free               |
| Vector DB      | Qdrant Cloud    | 1 GB free                      |
| Object storage | Cloudflare R2   | 10 GB free                     |
| Observability  | Grafana Cloud   | Free tier (traces + logs)      |

Fallback if any free tier changes: Contabo VPS 4 vCPU/8 GB (~$4.50/mo).

**Rationale**: Portfolio project with no revenue must be free to run.

**Consequences**:
- (+) Zero cost baseline.
- (–) Multiple providers to monitor; secrets management across all.

**Cannot change without**: New ADR.

---

## ADR-025: Product language — bilingual EN + ID

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21

**Decision**: UI, documentation of contracts, and analysis outputs
support English and Bahasa Indonesia. LLM prompts written in
English; instruct model to detect input language and respond in
same language unless overridden. Golden dataset for evals includes
both languages equally.

Language of code, comments, commit messages, ADRs, `learning_docs/`:

- Repo-public content: English (portfolio-facing).
- `learning_docs/` (private repo): Indonesian OK.

**Rationale**: Serves Indonesian legal market as portfolio narrative;
demonstrates multilingual AI-native design.

**Consequences**:
- (+) Real-world relevance for BRI-adjacent market.
- (–) Doubles the eval dataset effort; prompt tuning must cover both.

**Cannot change without**: New ADR.

---

## ADR-026: LLM strategy — free-tier first, paid tier opt-in

**Status**: Accepted &nbsp;·&nbsp; **Date**: 2026-07-21 &nbsp;·&nbsp; **Supersedes**: ADR-012

**Context**: ADR-012 defined a split strategy (Gemini in dev, Claude
Haiku in prod). During implementation review we reprioritized cost:
zero-revenue portfolio project should not require any paid provider
to run in production. Paid providers may be enabled as an optional
quality upgrade.

**Options considered**:
1. Keep ADR-012 as-is (paid default in prod) — rejected: contradicts
   ADR-024 free-tier posture.
2. Free-tier for everything, remove Claude entirely — rejected:
   loses portfolio narrative of multi-provider abstraction and
   caps quality ceiling.
3. **Free-tier default in every env, paid opt-in via env var — chosen.**

**Decision**: `packages/llm` routes per env var `LLM_TIER=free|paid`
(default `free`). Per-task override via
`CIQ_LLM_OVERRIDE_<CATEGORY>=<provider>` remains from ADR-012.

**Free tier (default)** — every environment:

| Category            | Primary            | Fallback              |
| ------------------- | ------------------ | --------------------- |
| `extract`           | Gemini 2.0 Flash   | Groq (Llama 3.3 70B)  |
| `classify`          | Gemini 2.0 Flash   | Groq                  |
| `risk_assessment`   | Gemini 2.0 Flash   | Groq                  |
| `summary`           | Gemini 2.0 Flash   | Groq                  |
| `qa`                | Gemini 2.0 Flash   | Groq                  |
| `compare`           | Gemini 2.0 Flash   | Groq                  |
| `redline`           | Gemini 2.0 Flash   | Groq                  |
| `search_rewrite`    | Gemini 2.0 Flash   | Groq                  |
| `contextualize`     | Gemini 2.0 Flash   | Groq                  |
| `judge`             | Groq (Llama 3.3)   | Gemini 2.0 Flash      |

Judge intentionally routes to Groq (different provider from
generation) to preserve the R300.8 intent that judge ≠ generator.
The fallback direction inverts for `judge`.

**Paid tier (opt-in via `LLM_TIER=paid`)**:

| Category            | Primary            | Fallback              |
| ------------------- | ------------------ | --------------------- |
| `extract`           | Gemini 2.0 Flash   | Groq                  |
| `classify`          | Gemini 2.0 Flash   | Groq                  |
| `risk_assessment`   | Claude Haiku 4.5   | Gemini 2.0 Flash      |
| `summary`           | Claude Haiku 4.5   | Gemini 2.0 Flash      |
| `qa`                | Claude Haiku 4.5   | Gemini 2.0 Flash      |
| `compare`           | Claude Haiku 4.5   | Gemini 2.0 Flash      |
| `redline`           | Claude Haiku 4.5   | Gemini 2.0 Flash      |
| `search_rewrite`    | Gemini 2.0 Flash   | Groq                  |
| `contextualize`     | Gemini 2.0 Flash   | Groq                  |
| `judge`             | Claude Haiku 4.5   | Gemini 2.0 Flash      |

Quality-sensitive tasks route to Claude; high-volume, cheap tasks
stay on Gemini to keep cost bounded.

**Rationale**:
- Zero baseline cost, aligned with ADR-024.
- Portfolio narrative preserved: provider abstraction is
  demonstrable via the tier switch itself, not by mandating paid.
- Paid tier remains a one-env-var toggle for demo days or
  benchmarking; not a code change.
- Golden dataset (R300.9) can run under either tier to compare
  quality vs cost tradeoff — this itself is a `learning_docs/`
  artifact worth having.

**Consequences**:
- (+) Anyone can clone and run the full stack for $0.
- (+) Quality upgrade path exists without refactor.
- (–) Quality thresholds in R300.9 must be calibrated against Gemini
  Flash baseline, not Claude Haiku. If a threshold is unreachable on
  free tier, adjust threshold (via ADR) or scope task-category
  quality expectations, not silently switch tiers.
- (–) Groq free tier has strict rate limits; if hit, fallback loops
  back to Gemini — potential circular fallback avoided by never
  fallback-of-fallback (single hop only, per R500.21).
- (–) Rate limits on both Gemini and Groq mean paid tier may still
  be needed for load testing (R300.16) at scale.

**Migration from ADR-012**:
- No code migration; `packages/llm` was never implemented against
  the old matrix (implementation begins at T-020).
- T-020, T-021 file specs unchanged (adapters exist regardless of
  routing).
- R500.3 table (Per-Task Model Selection) updated in same commit
  as this ADR.

**Cannot change without**: New ADR.

---

## How To Add A New ADR

1. Draft the ADR at the bottom of this file with next available ID.
2. Add a row to the Index table.
3. Update any rule (`.harness/rules/R###`) that references the
   superseded decision, in the same commit.
4. Commit with `docs: ADR-### <title>` in the commit message.
5. If the ADR changes existing behavior in code, follow up with a
   `refactor:` commit that brings code into alignment.

Never edit or delete an accepted ADR. To reverse a decision,
add a new ADR with `Supersedes: ADR-###` and mark the old one
`Status: Superseded by ADR-###`.
