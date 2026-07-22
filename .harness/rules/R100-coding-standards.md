# R100 — Coding Standards

Rules governing how TypeScript code is written across the monorepo.
Every rule states MUST / MUST NOT / SHOULD / SHOULD NOT / MAY per
R000.7, with positive and negative examples.

## Scope

All `.ts` and `.tsx` files in `apps/*` and `packages/*`. Excludes
generated files (`packages/db/generated/`, `.next/`, `dist/`) and
third-party vendored code.

---

## R100.1 — TypeScript Strictness

**MUST**: extend the shared strict config.

```json
// tsconfig.json in any package
{
  "extends": "@contractiq/config/tsconfig/strict.json"
}
```

Shared strict config sets:

```json
{
  "strict": true,
  "noUncheckedIndexedAccess": true,
  "exactOptionalPropertyTypes": true,
  "noFallthroughCasesInSwitch": true,
  "noImplicitOverride": true,
  "forceConsistentCasingInFileNames": true,
  "isolatedModules": true
}
```

**MUST NOT**: relax any of these flags in a package-level override.
If a legitimate case exists, escalate via ADR.

## R100.2 — No `any`

**MUST NOT**: use the `any` type in any position.

### Violation
```ts
function parse(input: any): any {
  return JSON.parse(input);
}
```

### Correct
```ts
function parse(input: string): unknown {
  return JSON.parse(input);
}

// At the call site, narrow with zod:
const parsed = ContractSchema.parse(parse(raw));
```

**Escape hatches**:

- `unknown` — for values whose shape is not known at compile time.
  Narrow with a zod schema or a type guard before use.
- `never` — for exhaustive switch checks and unreachable branches.

If a third-party library forces `any` in a signature, wrap it once
at the boundary and expose a typed API:

```ts
// packages/api/src/vendor-shim.ts
import { poorlyTypedLib } from 'legacy-lib';

export function typedCall(input: TypedInput): TypedOutput {
  return poorlyTypedLib(input as unknown as never) as TypedOutput;
}
// waives: R100.2 — vendor boundary, contained to this shim.
```

## R100.3 — No `@ts-ignore`, No `@ts-nocheck`

**MUST NOT** use `@ts-ignore` or `@ts-nocheck`.

**MAY** use `@ts-expect-error` on a single line, immediately above
the line it suppresses, with a comment explaining why. `@ts-expect-error`
is preferred because it fails when the error goes away — preventing
stale suppressions.

### Correct
```ts
// @ts-expect-error — third-party types missing for X; tracked in issue #42
someCall(argWithWrongType);
```

## R100.4 — Runtime Validation at Trust Boundaries

**MUST**: validate with zod at every trust boundary:

- HTTP request body, query, params, headers (Fastify's zod type
  provider).
- Environment variables (parse `process.env` into a zod-validated
  config object at startup).
- External API responses (LLM providers, third-party HTTP APIs).
- Data read from Redis (BullMQ job payloads, cache reads).
- Data read from R2 (only if the shape matters to the caller).

**MUST NOT**: cast unknown into a typed shape without validation.

### Violation
```ts
const body = req.body as UploadContractDto;  // ❌
```

### Correct
```ts
const body = UploadContractSchema.parse(req.body);
```

Prisma reads are exempt — schema.prisma is the source of truth
and Prisma Client types are trustworthy.

## R100.5 — Naming

**Files & directories**:
- Modules: `kebab-case` (`extract-clauses.ts`).
- React components: `PascalCase.tsx` (`RiskBadge.tsx`).
- Test files: mirror source with `.test.ts` (`extract-clauses.test.ts`).
- Storybook: `.stories.tsx`.

**Symbols**:
- Types & interfaces: `PascalCase` — no `I` prefix.
- Type aliases: `PascalCase`.
- Constants: `SCREAMING_SNAKE_CASE` only for module-scoped
  compile-time constants. Runtime config is `camelCase`.
- Enums: avoid; prefer discriminated unions with string literals.
- Functions & variables: `camelCase`.
- React components & hooks: `PascalCase` for components,
  `useCamelCase` for hooks.
- Booleans: `is/has/can/should` prefix (`isLoading`, `hasAccess`).

### Violation
```ts
interface IContract { ... }        // ❌ R100.5 (I prefix)
const maxRetries = 3;              // in module scope, const — should be SCREAMING
enum RiskLevel { LOW, MED, HIGH }  // ❌ prefer discriminated union
```

### Correct
```ts
interface Contract { ... }
const MAX_RETRIES = 3;
type RiskLevel = 'low' | 'medium' | 'high';
```

## R100.6 — Import Order

Enforced by Biome. Groups separated by a blank line, in this order:

1. Node built-ins (`node:*`).
2. External packages (`react`, `zod`, `@radix-ui/*`).
3. Workspace packages (`@contractiq/*`).
4. Relative parent (`../`).
5. Relative same-directory (`./`).
6. Type-only imports (last group, sorted like above).

**MUST**: use `import type` for type-only imports so `isolatedModules`
transpilation is clean.

### Correct
```ts
import { readFile } from 'node:fs/promises';

import { z } from 'zod';
import { FastifyInstance } from 'fastify';

import { db } from '@contractiq/db';
import { ContractSchema } from '@contractiq/shared';

import { computeRisk } from '../services/risk';
import { validateInput } from './validate';

import type { Contract } from '@contractiq/db';
```

## R100.7 — Async & Promise Discipline

**MUST**:
- `await` every returned promise unless intentionally fire-and-forget.
- Handle promise rejections — no unhandled promise rejections
  reaching the event loop.
- Use `Promise.all` for independent concurrent work.
- Use `Promise.allSettled` when partial success is meaningful.

**MUST NOT**:
- Use `.then().catch()` chains in new code — prefer `async/await`.
- Fire-and-forget a promise without `void` prefix or comment
  explaining why.

### Violation
```ts
async function analyze(id: string) {
  saveToDb(id);  // ❌ ignored promise
  return computeResult();
}
```

### Correct
```ts
async function analyze(id: string) {
  await saveToDb(id);
  return computeResult();
}

// Fire-and-forget with explicit signal:
void reportMetric('analysis_started', { id });
// waives: R100.7 — metric emission is best-effort, non-blocking.
```

## R100.8 — Error Handling

**MUST**:
- Throw typed errors that extend a base `AppError` class.
- Include an error `code` (stable string identifier) on every
  domain error.
- Preserve original errors via `cause` when re-throwing.

**MUST NOT**:
- Throw string literals or plain objects.
- Swallow errors silently with empty `catch` blocks.
- Use exceptions for control flow (e.g., "throw to return early").

### Base error contract
```ts
// packages/shared/src/errors.ts
export class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public cause?: unknown,
    public meta?: Record<string, unknown>,
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class NotFoundError extends AppError { ... }
export class ValidationError extends AppError { ... }
export class ExternalServiceError extends AppError { ... }
```

### Violation
```ts
try {
  await callLLM();
} catch (e) {
  // silent
}
```

### Correct
```ts
try {
  await callLLM();
} catch (e) {
  throw new ExternalServiceError(
    'LLM_UNAVAILABLE',
    'Failed to reach LLM provider',
    e,
    { provider: 'gemini', operation: 'complete' },
  );
}
```

## R100.9 — Immutability

**MUST**:
- Prefer `const` over `let`.
- Prefer non-mutating array/object methods (`map`, `filter`, spread)
  over mutating ones (`push`, `sort` in place, `Object.assign` to
  existing objects).

**MAY** use `let` when reassignment genuinely simplifies the code
(loop accumulators, incremental parsers) — comment briefly why.

**MUST NOT**:
- Mutate function parameters.
- Mutate exported constants.

### Violation
```ts
function normalize(items: Item[]) {
  items.sort((a, b) => a.order - b.order);  // ❌ mutates input
  return items;
}
```

### Correct
```ts
function normalize(items: Item[]) {
  return [...items].sort((a, b) => a.order - b.order);
}
```

## R100.10 — Function Length & Complexity

**SHOULD**:
- Functions ≤ 40 lines. If longer, extract sub-functions with
  descriptive names.
- Cyclomatic complexity ≤ 10. Enforced by Biome.
- Max 4 parameters. If more, use an object parameter with a
  named type.

**MAY** exceed these limits when the function is a straight-line
transform (e.g., a large `switch` with no logic branching), or
when extraction would harm readability. Comment with rationale.

## R100.11 — Module Boundaries (Bounded Contexts)

**MUST NOT**: cross-import a module's internal folders. Import only
from the module's public entry (`index.ts`).

### Violation
```ts
// In apps/api/src/modules/analysis/services/summary.ts
import { ContractRepo } from '../../contracts/repositories/contract-repo';  // ❌
```

### Correct
```ts
import { ContractRepo } from '../../contracts';
// where contracts/index.ts explicitly re-exports ContractRepo
```

Enforcement: dependency-cruiser rule blocks cross-internal imports
in CI.

## R100.12 — No Raw DB Queries Outside Repositories

**MUST**: use repository classes for all DB access.
**MUST NOT**: call `db.<model>.findMany` or raw Prisma methods in
route handlers, services, or workers.

**MAY**: use raw SQL when Prisma can't express the query, but
only inside a repository method and with a comment explaining why.

Rationale: tenant scoping (ADR-011) is enforced in the repository
base class. Bypassing the repo bypasses the enforcement.

## R100.13 — Config from Env, Validated Once at Startup

**MUST**:
- Parse `process.env` into a typed `Config` object at startup via zod.
- Fail fast if required env is missing or malformed.
- Import `config` (the parsed object), never `process.env` directly
  in application code.

### Violation
```ts
const key = process.env.CLAUDE_API_KEY;
if (key) {
  callClaude(key);
}
```

### Correct
```ts
// apps/api/src/config.ts
export const config = ConfigSchema.parse(process.env);

// elsewhere
import { config } from '../config';
callClaude(config.claudeApiKey);
```

## R100.14 — Logging

**MUST**:
- Use the shared `logger` from `@contractiq/shared/logger` (pino
  under the hood).
- Log at appropriate levels: `debug`, `info`, `warn`, `error`.
- Include structured context (`request_id`, `workspace_id`,
  `user_id` where available).

**MUST NOT**:
- Use `console.log`, `console.error`, `console.warn` in
  application code (allowed in scripts and tests).
- Log secrets, PII, or full LLM inputs/outputs in production
  (redaction is default; opt-in for local dev only).

### Violation
```ts
console.log('User signed up:', user);  // ❌ level + PII risk
```

### Correct
```ts
logger.info({ userId: user.id, workspaceId: user.workspaceId }, 'user_signed_up');
```

## R100.15 — Comments

**MUST**:
- Comment **why**, not **what**. Code should read as its own "what".
- Comment non-obvious business rules with a link to the ADR or the
  rule number (`R###`).
- Comment TODOs with an issue reference: `// TODO(#123): ...`.
  Unreferenced TODOs are blocked by CI.

**MUST NOT**:
- Leave commented-out code. Delete it — git preserves history.
- Write comments that duplicate the code (`// increment i`).

### Correct
```ts
// R500.4 — LLM calls carry tenant context in span attributes so
// cross-tenant cost allocation is possible.
span.setAttribute('workspace_id', ctx.workspaceId);
```

## R100.16 — File Length

**SHOULD**: files ≤ 300 lines. Longer files signal a missing
abstraction.

**MAY** exceed for generated files, large schema definitions,
route registrations, or config — comment at top of file
explaining why.

## R100.17 — No Circular Dependencies

**MUST NOT** introduce cyclic imports between modules or packages.
Enforced by dependency-cruiser in CI.

If a cycle emerges as a refactor by-product, resolve by extracting
the shared part into `packages/shared` or a lower-level module —
never by restructuring imports lazily.

## R100.18 — Public vs Internal Exports

**MUST**: mark internal helpers as such via file location — inside
a module, only `index.ts` re-exports the public API. Other files
are considered internal.

**SHOULD**: use `_` prefix on file names for internals that must
sit at the module root (rare): `_helpers.ts`. This is a signal;
enforcement is at import boundaries via R100.11.

## R100.19 — No Default Exports (with narrow exceptions)

**MUST NOT** use `export default` in application code.
**MAY** use `export default` where the framework requires it
(Next.js pages, Storybook stories, some tool configs).

Rationale: named exports make refactors and IDE renames safer;
default exports encourage inconsistent naming at call sites.

## R100.20 — React-Specific Rules

Applies to `.tsx` in `apps/web` and `packages/ui`.

**MUST**:
- Use function components. No class components.
- Use hooks per React's rules-of-hooks (enforced by Biome plugin).
- Extract hooks that grow beyond ~30 lines into named custom hooks.
- Type props explicitly. No `React.FC` (it hides prop inference).

### Correct
```tsx
type Props = {
  contractId: string;
  onSelect?: (id: string) => void;
};

export function ContractCard({ contractId, onSelect }: Props) {
  ...
}
```

**MUST NOT**:
- Use `React.FC` or `React.FunctionComponent`.
- Inline handler creation in JSX props when the handler is
  non-trivial (extract with `useCallback` if identity matters,
  or a named function).
- Access `window`, `document`, `localStorage` without an
  isomorphic guard (SSR safety) — extract into `useEffect` or a
  hook that checks `typeof window`.

## R100.21 — Node.js / Server-Specific Rules

**MUST**:
- Handle SIGTERM and SIGINT gracefully (close DB, Redis, Qdrant
  connections; drain in-flight requests).
- Use `AbortController` for cancellable operations (HTTP requests,
  LLM streams, DB queries where supported).

**MUST NOT**:
- Use synchronous `fs.readFileSync`, `fs.writeFileSync`, etc.
  outside startup / build scripts.
- Use `process.exit()` inside route handlers or worker jobs —
  throw a typed error and let the framework handle it.

## R100.22 — Waiver Convention

When a rule must be broken deliberately, place a waiver comment
immediately above the offending line:

```ts
// waives: R100.N — <one-line rationale>
```

Waivers are grep-able. CI reports the count per PR; unbounded
growth triggers a review discussion (not a block).

Never chain waivers to cover a systemic issue — file an ADR
instead.

## R100.23 — CI Enforcement Summary

| Rule       | Tool                     | Gate               |
| ---------- | ------------------------ | ------------------ |
| R100.1     | tsc                      | blocking           |
| R100.2, .3 | Biome + `tsc`            | blocking           |
| R100.4     | Manual review + tests    | reviewer           |
| R100.5     | Biome (naming rules)     | blocking           |
| R100.6     | Biome (import sorter)    | blocking (autofix) |
| R100.7     | Biome (`no-floating-promises`) | blocking     |
| R100.8     | Manual review            | reviewer           |
| R100.9     | Biome (prefer-const, no-param-reassign) | blocking |
| R100.10    | Biome (complexity, max-params) | warn → block after 3 in a PR |
| R100.11    | dependency-cruiser       | blocking           |
| R100.12    | grep pattern in CI       | blocking           |
| R100.13    | Startup fail-fast + review | runtime + reviewer |
| R100.14    | Biome (no-console) + review | blocking + reviewer |
| R100.15    | grep for unreferenced TODO | blocking         |
| R100.16    | Biome max-lines          | warn               |
| R100.17    | dependency-cruiser       | blocking           |
| R100.18–.19 | Biome                   | blocking           |
| R100.20    | Biome + react plugin     | blocking           |
| R100.21    | Manual review            | reviewer           |
| R100.22    | grep count in CI comment | informational      |

The Biome config that implements the blocking gates lives in
`packages/config/biome/base.json`.
