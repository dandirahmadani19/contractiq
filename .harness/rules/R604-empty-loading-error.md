# R604 — Empty, Loading, and Error States

Every page, list, and data-fetching component MUST implement three
deliberate states: empty, loading, error. Missing any of them
blocks merge.

## R604.1 — Empty State Contract

Every empty state MUST have:

1. **Specific copy** — names what's absent and what to do next.
   Not "No data".
2. **One primary action** — the most likely thing the user came to
   do.
3. **Optional secondary hint** — link to docs or example.
4. **Optional illustration** — monochrome line art, no emoji, no
   stock. Skip if a simple icon suffices.

### Example (correct)
```tsx
<EmptyState
  title="No contracts yet."
  description="Upload your first contract to see clause-level analysis and risk scoring."
  primaryAction={<Button>Upload contract</Button>}
  secondaryAction={<Link href="/docs/quickstart">Read the quickstart →</Link>}
  illustration={<UploadIllustration />}
/>
```

### Example (violation)
```tsx
<div>No data 🤷</div>  // ❌ R604.1
```

## R604.2 — Loading State Contract

**MUST** distinguish by expected latency:

| Latency         | Treatment                                      |
| --------------- | ---------------------------------------------- |
| < 200ms         | No loading UI; content pops in                 |
| 200ms – 2s      | Skeleton with layout matching final content   |
| 2s – 10s        | Skeleton + progress hint ("Analyzing 12 clauses...") |
| > 10s           | Background job pattern (SSE progress + non-blocking UI) |

**MUST NOT**:
- Full-page spinner for anything short of cold-start.
- Skeleton for content that arrives instantly.
- Silent long waits without progress.

## R604.3 — Skeleton Rules

- **MUST** match final layout: rectangle positions equal to final
  content positions. Not decorative shapes.
- **MUST** use `linear` shimmer motion (R603.8).
- **MUST NOT** contain text placeholders like "Loading...";
  skeleton is the signal.
- **MUST NOT** flash on rapid transitions — introduce with
  200ms delay if fetching may resolve immediately.

## R604.4 — Error State Contract

Every error state MUST have:

1. **Specific cause** — what failed. If the error is opaque, log
   backend and show a friendly summary with an error ID.
2. **Suggested action** — retry, contact support, check status.
3. **Preserved user input** — never wipe form state on error.
4. **Error ID** for user to quote (e.g., `err_2026_07_21_abc123`).

### Example (correct)
```tsx
<ErrorState
  title="Couldn't reach the analysis service."
  description="Retrying automatically. If this persists, quote error ID abc123 to support."
  primaryAction={<Button onClick={retry}>Retry now</Button>}
  errorId="err_2026_07_21_abc123"
/>
```

## R604.5 — Streaming Error (Q&A / Analysis)

When an LLM stream fails mid-response:
- **MUST** preserve the partial response.
- **MUST** append a specific inline error to the message
  ("Connection dropped. The rest of the answer wasn't generated.").
- **MUST** offer "Regenerate" that restarts from the same prompt.

## R604.6 — Storybook Coverage

Every component in `packages/ui` that has these states MUST have
stories: `Empty`, `Loading`, `Error`. Missing a story is a lint
violation via a custom Storybook check.

## R604.7 — Copy Voice

Error and empty state copy MUST:
- Use plain language, not technical jargon (unless the audience
  is developers, e.g. docs).
- Be honest about what went wrong. No "Oops!" or "Whoops!".
- Fit in 2 lines on standard screens.

MUST NOT:
- Use exclamation points.
- Anthropomorphize ("The server is taking a nap").
- Blame the user ("You entered invalid data" → "This value doesn't
  look right").
