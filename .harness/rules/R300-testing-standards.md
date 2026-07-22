# R300 — Testing Standards

Rules governing tests across the monorepo. Extends ADR-019.
Every non-trivial code change must satisfy at least one R300
sub-rule for coverage; every LLM-touching change must satisfy at
least one LLM-testing sub-rule (R300.7–R300.10).

## Scope

All test files (`*.test.ts`, `*.test.tsx`, `*.spec.ts`), Playwright
specs (`*.e2e.ts`), k6 scripts (`tooling/k6/*.js`), and
evaluation code (`packages/evals/**`).

---

## R300.1 — Testing Pyramid

**MUST**: distribute test effort per this shape.

```
       ╱ E2E ╲              < 5% of test count, ~30% of CI time budget
      ╱───────╲
     ╱ Contract ╲            < 10%
    ╱─────────────╲
   ╱  Integration  ╲         ~25%
  ╱─────────────────╲
 ╱      Unit         ╲       ~60%
╱─────────────────────╲
```

Plus, orthogonal to the pyramid:

- **LLM evals** — golden dataset + judges, run on demand and in CI
  when triggered by `ai:` label.
- **Load** — k6, run on demand and pre-release.

**MUST NOT**:
- Skip a layer to save time. Missing integration coverage is not
  compensated by extra E2E.
- Rely on E2E for logic that a unit test can cover.

Rationale: unit tests are cheap, fast, and localize failures.
E2E is expensive, slow, and locates but does not localize
failures. Integration is the middle ground where most bugs live.

## R300.2 — Unit Tests

**Scope**: pure functions, service methods, hooks, utility code,
schema validators.

**MUST**:
- Live next to the code: `foo.ts` → `foo.test.ts` in same folder.
- Use `Vitest`.
- Complete in ≤ 50ms per test file typical, ≤ 200ms hard cap.
- Use no network, no DB, no filesystem (in-memory only). Mock
  dependencies at the injection boundary (awilix container), not
  at the module level (`vi.mock` is a last resort).

**MUST NOT**:
- Depend on wall clock without freezing (`vi.useFakeTimers`).
- Share state between tests within a file (each `it()` starts
  from clean state via `beforeEach`).
- Depend on other test files running first.

### Example (correct)
```ts
// packages/llm/src/agent/tools/classify-clause.test.ts
import { describe, it, expect } from 'vitest';
import { buildClassifyPrompt } from './classify-clause';

describe('buildClassifyPrompt', () => {
  it('includes taxonomy definitions', () => {
    const prompt = buildClassifyPrompt({ clauseText: 'sample' });
    expect(prompt).toContain('liability');
    expect(prompt).toContain('confidentiality');
  });
});
```

## R300.3 — Integration Tests

**Scope**: interactions between two or more real components (DB
repositories, queue producers/consumers, service ↔ external
adapter).

**MUST**:
- Live in the module they primarily exercise:
  `apps/api/src/modules/contracts/repositories/contract-repo.integration.test.ts`.
- Use **Testcontainers** for real Postgres, Redis, Qdrant instances
  in Docker.
- Reset external state between tests (truncate tables, flush Redis,
  clear Qdrant collections).
- Complete in ≤ 5s per test typical, ≤ 30s hard cap.

**MUST NOT**:
- Hit real external SaaS APIs (LLM providers, R2, Qdrant Cloud,
  Neon). Those are for E2E or excluded from CI entirely.
- Use `Mockito`/`Sinon` for the primary components under test.
  Integration means "the real thing"; use mocks only for
  peripheral out-of-scope deps.

### Testcontainers helper
```ts
// packages/db/src/test/postgres-container.ts
import { PostgreSqlContainer } from '@testcontainers/postgresql';

export async function startPostgres() {
  const container = await new PostgreSqlContainer('postgres:16')
    .withDatabase('contractiq_test')
    .start();
  return {
    url: container.getConnectionUri(),
    stop: () => container.stop(),
  };
}
```

Single shared container per test file, torn down in `afterAll`.

## R300.4 — E2E Tests

**Scope**: user journeys through the deployed web app.

**MUST**:
- Live in `apps/web/e2e/*.e2e.ts`.
- Use **Playwright**.
- Run against a Vercel preview deployment for CI, or `pnpm dev`
  local for developer flow.
- Test happy paths and one representative failure path per feature.
- Complete the full suite in ≤ 5 minutes on CI.

**MUST NOT**:
- Cover every acceptance criterion at E2E level. E2E validates the
  wiring; unit/integration cover the logic.
- Use `sleep` / arbitrary waits. Use Playwright's auto-waiting or
  explicit `expect(...).toBeVisible()`.
- Test third-party dashboards (Vercel, Neon UI, Grafana).

### Example
```ts
// apps/web/e2e/contract-upload.e2e.ts
import { test, expect } from '@playwright/test';

test('upload NDA → see extracted clauses', async ({ page }) => {
  await page.goto('/');
  await signIn(page, testUser);
  await page.getByRole('button', { name: 'Upload contract' }).click();
  await page.setInputFiles('[data-testid="file-input"]', 'fixtures/nda-en.pdf');
  await expect(page.getByRole('heading', { name: /nda/i })).toBeVisible({ timeout: 60_000 });
  await expect(page.locator('[data-testid="clause-item"]')).toHaveCount(await page.locator('[data-testid="clause-item"]').count());
});
```

## R300.5 — Contract (API) Tests

**Scope**: HTTP endpoints — request/response shape, status codes,
error contracts.

**MUST**:
- Use zod schemas from `packages/shared` — same schemas the
  runtime enforces.
- Use Fastify's `.inject()` (in-process, no network).
- Verify happy path + at least: 400 validation error, 401 auth,
  404 not-found where applicable, 403 tenant boundary.

**MUST NOT**:
- Duplicate zod schemas in test files. Import from
  `packages/shared`.

### Example
```ts
// apps/api/src/modules/contracts/routes/upload.test.ts
import { describe, it, expect } from 'vitest';
import { buildApp } from '../../../test/build-app';
import { UploadContractResponseSchema } from '@contractiq/shared';

describe('POST /contracts', () => {
  it('202 returns valid response shape', async () => {
    const app = await buildApp({ authAs: testUser });
    const res = await app.inject({
      method: 'POST',
      url: '/contracts',
      payload: sampleMultipart,
    });
    expect(res.statusCode).toBe(202);
    expect(() => UploadContractResponseSchema.parse(res.json())).not.toThrow();
  });

  it('403 when workspace not in session', async () => {
    const app = await buildApp({ authAs: userWithoutWorkspace });
    const res = await app.inject({ method: 'POST', url: '/contracts', payload: sampleMultipart });
    expect(res.statusCode).toBe(403);
  });
});
```

## R300.6 — Tenant Isolation Tests (MANDATORY)

**MUST**: every module that handles tenant-scoped data has at
least one dedicated tenant-isolation test.

Test contract:

1. Set up **two workspaces**: A and B, each with data.
2. Assert read scoped to A returns **only** A's rows.
3. Assert write scoped to A **cannot mutate** B's rows.
4. Assert queries with **no context** return zero rows and
   writes reject.

### Example
```ts
// apps/api/src/modules/contracts/repositories/contract-repo.tenant.test.ts
import { describe, it, expect } from 'vitest';
import { withWorkspaceContext } from '../../../tenancy';
import { ContractRepo } from './contract-repo';

describe('ContractRepo tenant isolation', () => {
  it('read in workspace A returns only A rows', async () => {
    const inA = await withWorkspaceContext(workspaceA.id, () => ContractRepo.list());
    expect(inA.every(c => c.workspaceId === workspaceA.id)).toBe(true);
    expect(inA.some(c => c.workspaceId === workspaceB.id)).toBe(false);
  });

  it('update in workspace A cannot touch B row', async () => {
    await expect(
      withWorkspaceContext(workspaceA.id, () =>
        ContractRepo.update(contractInB.id, { title: 'hacked' })
      )
    ).rejects.toThrow();
    const check = await withWorkspaceContext(workspaceB.id, () => ContractRepo.get(contractInB.id));
    expect(check.title).not.toBe('hacked');
  });

  it('no context = empty reads, rejected writes', async () => {
    await expect(ContractRepo.list()).resolves.toEqual([]);
    await expect(ContractRepo.create({ title: 'x' })).rejects.toThrow();
  });
});
```

**MUST NOT**: skip this test category for "obviously safe" code.
The whole point is to catch non-obvious leaks.

Coverage requirement (R300.11): every repository extending
`TenantScopedRepository` has a matching `*.tenant.test.ts`.
CI grep-checks this.

## R300.7 — LLM VCR (Snapshot/Replay) Tests

**Scope**: any code that calls a real LLM provider.

**Concept**: record a real LLM response once, replay in CI. Like
tape-based mocking for HTTP but tuned for LLM I/O (large payloads,
streaming).

**MUST**:
- Use `packages/llm/src/test/vcr.ts` helper.
- Record cassettes in `packages/llm/cassettes/<test-name>.json`.
- Commit cassettes; they are review artifacts.
- Regenerate cassettes only when the prompt / tool schema
  intentionally changes. Regeneration is a separate PR that
  discloses the diff of the recorded output.

**MUST NOT**:
- Hit the real provider in CI (network + cost + flakiness).
- Silently regenerate cassettes without a PR that shows the diff.
- Use cassettes as the only test — pair with a semantic assertion
  (does the output make sense given the input?).

### Example
```ts
// packages/llm/src/agent/tools/extract-clauses.vcr.test.ts
import { describe, it, expect } from 'vitest';
import { withCassette } from '../../test/vcr';
import { extractClauses } from './extract-clauses';

describe('extract_clauses (VCR)', () => {
  it('extracts sections from NDA fixture', async () => {
    await withCassette('extract-clauses-nda-en.json', async () => {
      const result = await extractClauses({ text: fixtures.ndaEn });
      expect(result.clauses.length).toBeGreaterThanOrEqual(5);
      expect(result.clauses[0]).toMatchObject({
        id: expect.any(String),
        category: expect.any(String),
        text: expect.any(String),
      });
    });
  });
});
```

Cassette regeneration workflow:

```bash
# Manually, with explicit intent
VCR_MODE=record pnpm --filter @contractiq/llm test extract-clauses.vcr
git add packages/llm/cassettes/extract-clauses-nda-en.json
git diff --stat packages/llm/cassettes/  # show what changed
git commit -m "ai(llm): regenerate cassette after extract prompt change"
```

CI runs in `VCR_MODE=replay` (default). Missing cassette → test
fails with instructions.

## R300.8 — LLM-as-Judge Tests

**Scope**: qualitative properties of LLM output (faithfulness,
groundedness, style adherence) that can't be checked by schema
or regex.

**Concept**: use a second LLM call to grade the first LLM's output.
The judge returns a structured verdict; we assert on the verdict.

**MUST**:
- Use a **different model** for judging than for generation
  (reduces same-model bias). Recommended judge: `Claude Haiku` or
  `Gemini 2.0 Flash`.
- Use a **strict rubric** in the judge prompt with numeric or
  categorical output.
- Ship rubrics as versioned files in `packages/evals/rubrics/*.md`.
- Track judge agreement with human labels on a small validation
  set; if judge agreement drops below 80%, replace the judge (not
  the generation) and note in `learning_docs/`.

**MUST NOT**:
- Judge on unbounded free-text ("is this good?"). Rubric must
  decompose into scoreable dimensions.
- Compare model to itself.
- Use judge output as a hard CI gate without a golden benchmark
  (R300.9). Judge is expensive and non-deterministic.

### Example
```ts
// packages/evals/src/judges/summary-faithfulness.ts
import { z } from 'zod';
import { llm } from '@contractiq/llm';

const Verdict = z.object({
  faithfulness: z.number().min(0).max(1),
  hallucinations: z.array(z.string()),
  reasoning: z.string(),
});

export async function judgeSummary(source: string, summary: string) {
  const rubric = await loadRubric('summary-faithfulness');
  const result = await llm.complete({
    provider: 'claude-haiku',
    schema: Verdict,
    system: rubric,
    messages: [{ role: 'user', content: `Source:\n${source}\n\nSummary:\n${summary}` }],
  });
  return result;
}
```

## R300.9 — Golden Dataset Regression

**Scope**: end-to-end quality guarantees across LLM changes.

**Concept**: 50–100 curated (input, expected output) pairs.
Every LLM change reruns the dataset; a regression outside
threshold blocks merge.

**MUST**:
- Store golden dataset in `packages/evals/golden/<capability>/`.
  One capability per folder (`extract`, `classify`, `risk`, `qa`,
  `summary`, `compare`, `redline`, `search`).
- Every capability has at least: 10 EN + 10 ID fixtures (bilingual
  parity per ADR-025).
- Each entry has: `input.json`, `expected.json`, `notes.md`
  (why this case is in the dataset).
- Threshold per capability defined in
  `packages/evals/thresholds.json` (single source of truth).
- CI job `llm-eval` runs the dataset when PR label `ai:eval` set
  or when files under `packages/llm/`, `packages/evals/`, or
  prompt files change.
- Regression is a block; improvement is celebrated in the PR body.

**MUST NOT**:
- Grow the dataset with only easy cases. Aim for a distribution
  including edge cases (long contracts, missing sections, mixed
  language).
- Delete entries when they fail. Fix the model / prompt; keep the
  hard case as regression protection.

### Threshold file
```json
// packages/evals/thresholds.json
{
  "extract": {
    "clause_recall": { "min": 0.9, "delta_block_below": -0.02 },
    "clause_precision": { "min": 0.85, "delta_block_below": -0.02 }
  },
  "classify": {
    "macro_f1": { "min": 0.85, "delta_block_below": -0.03 }
  },
  "risk": {
    "spearman_correlation": { "min": 0.7, "delta_block_below": -0.05 }
  },
  "summary": {
    "faithfulness": { "min": 0.9, "delta_block_below": -0.02 }
  },
  "qa": {
    "groundedness": { "min": 0.9, "delta_block_below": -0.02 }
  }
}
```

- `min`: absolute floor; below this always blocks.
- `delta_block_below`: relative regression vs the last main-branch
  run that also blocks (even if still above min).

## R300.10 — Structured Output Property Tests

**Scope**: LLM outputs that must conform to a zod schema (per
ADR-014).

**Concept**: use `fast-check` to generate arbitrary valid input
and assert the LLM adapter's schema-repair mechanism always
returns valid output or a typed failure.

**MUST**:
- Every tool schema has a companion property test asserting:
  1. Valid input → valid output (schema-parseable).
  2. Malformed provider response → repair loop kicks in → valid
     output OR typed `SchemaValidationError` after N=2 retries.

**MUST NOT**:
- Assert on exact output content in property tests. Assert on
  shape and invariants only.

### Example
```ts
// packages/llm/src/agent/tools/extract-clauses.property.test.ts
import { it } from 'vitest';
import fc from 'fast-check';
import { ExtractClausesSchema } from '@contractiq/shared';
import { extractClauses } from './extract-clauses';

it.prop([fc.string({ minLength: 100, maxLength: 5000 })])(
  'output always matches schema or throws SchemaValidationError',
  async (text) => {
    try {
      const result = await extractClauses({ text });
      ExtractClausesSchema.parse(result); // must not throw
    } catch (e) {
      // acceptable outcome: typed error, not a random parse crash
      if (!(e instanceof SchemaValidationError)) throw e;
    }
  }
);
```

Property tests use replay cassettes (R300.7) or a mocked
provider — never live LLM calls.

## R300.11 — Coverage Targets

**MUST**:

| Layer / Package                    | Statement | Branch |
| ---------------------------------- | --------- | ------ |
| `packages/llm`                     | ≥ 90%     | ≥ 85%  |
| `packages/shared`                  | ≥ 95%     | ≥ 90%  |
| `packages/db` (repositories)       | ≥ 90%     | ≥ 85%  |
| `apps/api` (services + repos)      | ≥ 85%     | ≥ 80%  |
| `apps/api` (routes)                | ≥ 80%     | ≥ 70%  |
| `apps/web` (components in ui pkg)  | ≥ 80%     | ≥ 70%  |
| `apps/web` (pages)                 | best-effort — covered by E2E |
| Everything else                    | ≥ 80% overall             |

**MUST NOT**: game coverage by adding trivial tests (`expect(true).toBe(true)`).
Reviewer discretion catches these.

**MAY** waive per-file with a comment linking to a follow-up:
```ts
/* c8 ignore next 20 -- waives: R300.11 — see #142 */
```

Waivers count against the PR's total; too many triggers a review.

## R300.12 — Test File Organization

```
packages/llm/src/agent/tools/
├── classify-clause.ts
├── classify-clause.test.ts          # unit
├── classify-clause.vcr.test.ts      # LLM VCR
├── classify-clause.property.test.ts # property
```

**MUST**:
- Same folder as source under test.
- Suffix identifies category: `.test.ts` (unit), `.integration.test.ts`,
  `.tenant.test.ts`, `.vcr.test.ts`, `.property.test.ts`, `.e2e.ts`
  (E2E lives in `apps/web/e2e/`).

**MUST NOT**:
- Bury tests in a distant `__tests__/` folder.
- Mix test categories in one file.

## R300.13 — Test Naming

**MUST**: `describe` + `it` reads as a sentence.

### Correct
```ts
describe('extractClauses', () => {
  it('returns clauses with stable IDs', ...);
  it('preserves original text verbatim', ...);
  it('rejects inputs shorter than 100 characters', ...);
});
```

### Violation
```ts
describe('test extract', () => {
  it('test 1', ...);       // ❌ meaningless
  it('should work', ...);  // ❌ vague; and 'should' is verbose
});
```

**MUST NOT**: use `test.only`, `describe.only`, `it.only`, or
`skip` in committed code. Enforced by Biome.

## R300.14 — Fixtures and Factories

**MUST**:
- Static fixtures in `packages/evals/fixtures/` (contracts, edge
  cases) — versioned with the repo.
- Programmatic factories in `packages/db/src/test/factories/` —
  functions that create realistic entities with sensible defaults.

**MUST NOT**:
- Inline large fixtures in test files.
- Use `faker.js` for LLM tests (non-deterministic input = flaky
  golden). Use seeded factories with fixed seeds per test.

### Factory example
```ts
// packages/db/src/test/factories/contract.ts
import { ulid } from 'ulid';
import type { Contract } from '../../generated';

export function makeContract(overrides: Partial<Contract> = {}): Contract {
  return {
    id: ulid(),
    workspaceId: overrides.workspaceId ?? ulid(),
    title: 'Master Services Agreement — Acme × Widget Co',
    status: 'INDEXED',
    createdAt: new Date('2026-01-01T00:00:00Z'),
    updatedAt: new Date('2026-01-01T00:00:00Z'),
    ...overrides,
  };
}
```

Note the realistic default title per R601.10.

## R300.15 — Test Isolation

**MUST**:
- Each test file owns its DB state — no cross-file assumptions.
- Truncate tables between tests within a file (`beforeEach`) or
  use per-test transactions with rollback.
- Redis flushed between tests.
- Qdrant collection reset between tests (drop + recreate).

**MUST NOT**:
- Rely on test order within a file.
- Use `process.env` mutation in tests without restoration.
- Share module-level state between test files (Vitest runs files
  in isolated workers; do not rely on this to hide state bugs).

## R300.16 — Load Testing

**Scope**: performance validation before releases and after
significant changes to hot paths (upload, embed, search, chat
streaming).

**MUST**:
- Use `k6`.
- Scripts in `tooling/k6/*.js`.
- Report artifacts to `learning_docs/measurements/` after every
  run (per R601.11, quantitative claims require sourced numbers).
- Include thresholds in the script; fail the run if unmet.

**MUST NOT**:
- Load-test third-party endpoints (they will rate-limit or bill).
- Run load tests against production without a maintenance window.

### Example
```js
// tooling/k6/search-load.js
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 20,
  duration: '2m',
  thresholds: {
    http_req_duration: ['p(95)<3000'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  const res = http.post('https://preview.contractiq.app/search',
    JSON.stringify({ query: 'unlimited liability' }),
    { headers: { 'Authorization': `Bearer ${__ENV.TOKEN}` } }
  );
  check(res, { '200': (r) => r.status === 200 });
}
```

## R300.17 — Flakiness Policy

**MUST**:
- Any test that fails intermittently in CI is quarantined within
  24 hours (marked with `.skip` and a `// FLAKY: <link to issue>` comment,
  with a filed issue).
- Quarantined tests are fixed or deleted within 1 week. No
  indefinite skip.

**MUST NOT**:
- "Retry" flaky tests to make CI green. Retries mask real races
  and Heisenbugs.
- Leave a flaky test in the suite without quarantine — it erodes
  trust in CI.

Flaky tests are code smells for concurrency bugs, order dependence,
or wall-clock timing. Treat them as such.

## R300.18 — Regression Test Convention

When fixing a bug:

**MUST**:
1. First write a failing test that reproduces the bug.
2. Then fix the code so the test passes.
3. Commit test and fix together (per R200.8, one logical change
   = one commit).

**MUST NOT**:
- Fix a bug without a regression test unless the fix is trivially
  observable (e.g., typo in copy).

Regression test naming convention:
```ts
it('regression: #<issue-number> — <one-line summary>', ...)
```

## R300.19 — Test-Only Code Hygiene

**MUST**:
- Test utilities live in `src/test/` within a package, or
  `packages/db/src/test/` for shared factories.
- Test utilities are excluded from production build
  (`tsconfig.build.json` excludes them).

**MUST NOT**:
- Import test utilities from production code.
- Expose "test-only" branches in production code (e.g.,
  `if (process.env.NODE_ENV === 'test') { ... }`) — refactor to
  DI so the test injects a fake.

## R300.20 — CI Enforcement Summary

| Rule           | Tool                                          | Gate       |
| -------------- | --------------------------------------------- | ---------- |
| R300.1         | Reviewer + coverage report                    | reviewer   |
| R300.2         | Vitest (default suite)                        | blocking   |
| R300.3         | Vitest (`--project=integration`)              | blocking   |
| R300.4         | Playwright                                    | blocking   |
| R300.5         | Vitest (contract subset)                      | blocking   |
| R300.6         | Grep: every `TenantScopedRepository` has `.tenant.test.ts` | blocking |
| R300.7         | Vitest with `VCR_MODE=replay`                 | blocking   |
| R300.8         | On-demand, PR label `ai:eval`                 | conditional|
| R300.9         | On-demand + triggered by prompt/eval changes  | conditional|
| R300.10        | Vitest (property subset)                      | blocking   |
| R300.11        | `c8` coverage report + thresholds             | blocking   |
| R300.12–.14    | Biome + directory-check script                | blocking   |
| R300.15        | Runtime (test isolation failure = fail loudly)| runtime    |
| R300.16        | On-demand + pre-release                       | manual     |
| R300.17        | Reviewer + weekly audit                       | reviewer   |
| R300.18        | Reviewer                                      | reviewer   |
| R300.19        | Biome (`no-restricted-imports` from test paths)| blocking  |

The Vitest projects config lives in `packages/config/vitest/`.
The eval trigger lives in `.github/workflows/llm-eval.yml`.
