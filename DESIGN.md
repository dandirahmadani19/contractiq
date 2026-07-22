# DESIGN.md — Design Principles & Tokens

**Status**: Living document. Amendments via ADR.

This document describes the **design language** of ContractIQ.
Its enforceable form lives in `.harness/rules/R600..R605`. Read
this file to understand the intent; consult the rules to check a
specific decision.

## Reading Order

1. [Design Philosophy](#design-philosophy)
2. [Mood & References](#mood--references)
3. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
4. [Design Tokens](#design-tokens)
5. [Typography](#typography)
6. [Color System](#color-system)
7. [Spacing & Layout](#spacing--layout)
8. [Motion](#motion)
9. [Iconography](#iconography)
10. [Component Composition](#component-composition)
11. [Empty / Loading / Error States](#empty--loading--error-states)
12. [Density Philosophy](#density-philosophy)
13. [Dark Mode](#dark-mode)
14. [Accessibility Baseline](#accessibility-baseline)

---

## Design Philosophy

Three principles, ranked by priority. When they conflict, higher
rank wins.

**1. Editorial-serious over playful-generic.** ContractIQ handles
legal documents. Users are professionals doing high-stakes work.
Every visual choice should read as "considered", "trustworthy",
"grounded" — not "cheerful startup". Serif type, restrained color,
information density.

**2. Density over airiness.** B2B power users are paid for speed
and accuracy. Give them information per screen, not decoration.
Whitespace exists for hierarchy, not for aesthetic. When in
doubt, show more, in more legible arrangements.

**3. Distinctive over familiar.** Recognizable defaults —
shadcn/ui look, Inter everywhere, `rounded-2xl`, purple-blue
gradients — signal "AI-generated". Distinctiveness comes from
one considered typography choice, one accent color used sparingly,
disciplined spacing, and empty/loading/error states that were
actually designed.

---

## Mood & References

ContractIQ sits at the intersection of three moods:

| Reference     | What we borrow                             |
| ------------- | ------------------------------------------ |
| **Linear**    | Dark-mode density, keyboard-first ergonomics, restraint in animation |
| **Anthropic** | Serif display type, editorial mood, calm palette |
| **Ironclad**  | Legal-professional trust signals, clause-first mental model |
| **Stripe Docs** | Information hierarchy in dense reference material |

Studied but **not** copied. The visual output should feel akin to
these — not derivative of any one.

Anti-references (do not draw from):

- Any AI-startup landing page from 2024 that used purple gradients
  and sparkle emoji.
- Default shadcn/ui showcase pages.
- SaaS marketing pages with a 3-feature-card layout, hero + CTA
  every 400px.

---

## Anti-Patterns to Avoid

Concrete list — every item is a MUST NOT enforced by R601. These
are the patterns that mark UI as "AI-generated" and this project
explicitly rejects them.

**Visual patterns to reject**:

1. Purple-to-blue gradients (of any orientation).
2. "AI-powered ✨" or any sparkle emoji in copy or icons.
3. Sparkles / stars / rocket emoji anywhere in the UI.
4. `rounded-2xl` or larger applied uniformly to every element.
5. Gradient text on headings (any gradient).
6. Glassmorphism backgrounds without a functional reason.
7. Framer Motion default `whileHover={{ scale: 1.05 }}` on
   everything.
8. Emoji used as icons (use Phosphor per ADR-018).
9. Hero + 3 feature cards + CTA layout with symmetric centering.
10. Default Tailwind Slate / Zinc palette (both are overused;
    we use Radix `sand` per ADR-016).

**Copy patterns to reject**:

11. "Transform your workflow with AI"
12. "Powered by cutting-edge AI"
13. "Revolutionize the way you work"
14. Lorem ipsum in any state (mock data must be real-shaped:
    "Vendor Master Services Agreement", not "Product Foo Bar").
15. Numeric claims without evidence ("10x faster", "99% accurate")
    unless the number came from an actual measurement stored in
    `learning_docs/`.

**Interaction patterns to reject**:

16. Every button, card, and icon animating on hover.
17. Toasts stacking without a stagger or lifespan.
18. Modals opening from center with a fade-and-scale that looks
    identical to every dashboard on Dribbble.
19. Skeleton loaders on content that arrives in <200ms (creates
    perceived slowness).
20. Loading spinners with no context ("Loading..." without saying
    what).

The full enforceable list lives in R601.

---

## Design Tokens

Tokens are the shared vocabulary between design and code. Every
value in a component comes from a token, not a literal.

Structure (in `packages/ui/src/tokens/`):

```
tokens/
├── color.ts         # Semantic + palette (from Radix Colors)
├── typography.ts    # Font families, sizes, weights, tracking
├── spacing.ts       # 4pt grid scale
├── radius.ts        # Border radius scale
├── shadow.ts        # Elevation scale
├── motion.ts        # Duration, easing curves
└── z-index.ts       # Layering scale
```

Consumed by:

- Tailwind theme (`tailwind.config.ts`) — automatic mapping.
- CSS variables (`:root` and `[data-theme]`) for runtime theming.
- Component props (typed against token unions, not strings).

Full token definitions live in R600.

---

## Typography

Three fonts, distinct roles per ADR-017.

| Role                | Font              | When                                          |
| ------------------- | ----------------- | --------------------------------------------- |
| **Display / heading** | Instrument Serif  | H1, H2, hero copy, section titles above H3    |
| **Body / UI**        | Inter             | H3 and below, paragraphs, buttons, inputs, nav |
| **Data / mono**      | Geist Mono        | Contract IDs, timestamps, code, numbers in dense tables, keyboard shortcuts |

**Instrument Serif rules**:
- Italic variant is the emphasis mode (not bold), giving editorial feel.
- Only for headings that lead a page or major section — not every H3.
- Optical size: use the display variant at sizes ≥ 32px; regular below.
- Never all-caps.

**Inter rules**:
- OpenType `ss01` feature enabled (single-story `a`) — small but
  distinctive touch.
- OpenType `cv11` for slashed zero when used in mixed-with-mono contexts.
- Weights used: 400 (body), 500 (UI emphasis), 600 (labels).
  Never 700+.

**Geist Mono rules**:
- Only for machine-generated content and interactive keyboard hints.
- Never for prose.
- Feature `zero` (slashed zero) always on.

Typography scale (from R602):

```
display-2xl  56 / 60  Instrument Serif Regular    tracking -0.02em
display-xl   40 / 46  Instrument Serif Regular    tracking -0.02em
display-lg   32 / 40  Instrument Serif Regular    tracking -0.01em
heading-lg   24 / 32  Inter Semibold              tracking -0.01em
heading-md   18 / 26  Inter Semibold              tracking  0
heading-sm   16 / 24  Inter Semibold              tracking  0
body-lg      16 / 26  Inter Regular               tracking  0
body-md      14 / 22  Inter Regular               tracking  0
body-sm      13 / 20  Inter Regular               tracking  0
caption      12 / 18  Inter Medium                tracking  0.01em
mono-md      13 / 20  Geist Mono Regular          tracking  0
mono-sm      12 / 18  Geist Mono Regular          tracking  0
```

All self-hosted via `@next/font` local — no Google Fonts CDN in
production (privacy + performance).

---

## Color System

Radix Colors, 12-step scales, OKLCH-based (ADR-016).

**Base neutral: `sand`.** Warm neutral, less mainstream than
`slate` or `zinc`. Reads as "paper" — apt for a contract app.

**Accent: `iris`.** Blue-violet, sober. Used sparingly for
interactive affordance and single points of emphasis. Never as
gradient. Never as background wash.

**Semantics** (all Radix):
- `red` — destructive, error
- `green` — success, positive risk (low risk = good)
- `amber` — warning, medium risk
- `crimson` — high risk (distinct from destructive red)

Step usage across the 12-step scale (Radix convention):

| Step | Use                                        |
| ---- | ------------------------------------------ |
| 1    | App background                             |
| 2    | Subtle background (side panels, code)      |
| 3    | UI element background (hover on 1)         |
| 4    | Hovered UI element background              |
| 5    | Active / selected UI element background    |
| 6    | Subtle borders, non-interactive separators |
| 7    | UI element borders and focus rings         |
| 8    | Hovered UI element borders                 |
| 9    | Solid backgrounds (buttons, badges)        |
| 10   | Hovered solid backgrounds                  |
| 11   | Low-contrast text                          |
| 12   | High-contrast text                         |

No arbitrary hex values in components. If a semantic doesn't
exist, add a token first.

---

## Spacing & Layout

**Grid**: 4pt base. All spacing values are multiples of 4.

**Scale** (Tailwind aligned):
```
xs   4    sm  8    md  12   lg  16
xl  24   2xl 32   3xl 48   4xl 64   5xl 96
```

**Container width**: max content width `1280px` for app pages,
`720px` for reading-heavy pages (contract text, docs). Never
`max-w-7xl` uncritically.

**Layout primitives**: use CSS Grid + Container Queries. Component
responsiveness responds to **parent width**, not viewport — matters
for panels in split views.

**Density modes**: MVP has one density. Future: `comfortable` /
`compact` toggle for power users. Design components with room to
support this without rewrite (spacing tokens, not literals).

---

## Motion

Restraint is the point. Motion communicates state change, not
mood. If a motion isn't communicating something specific, it
should not be there.

**Duration scale**:
- `instant` — 0ms (no transition, for focus rings)
- `fast` — 120ms (hover feedback, focus)
- `base` — 180ms (dropdown, tooltip)
- `slow` — 240ms (dialog open, drawer)
- `slower` — 320ms (layout transitions)

**Easing**:
- `swift` — `cubic-bezier(0.32, 0.72, 0, 1)` — default, spring-like
- `smooth` — `cubic-bezier(0.4, 0, 0.2, 1)` — for larger movement
- `linear` — only for continuous animations (progress, streaming)

**Never**:
- `whileHover={{ scale: 1.05 }}` as default (over-used AI pattern).
- Bounces on modal open.
- Motion on more than one property simultaneously except when
  purposeful (e.g., dialog: opacity + transform Y).

**Always**:
- Respect `prefers-reduced-motion` (motion library or media query).
- View Transitions API for cross-page navigation (Next.js Pages
  Router supports this via manual instrumentation).

Detail: R603.

---

## Iconography

Phosphor Icons per ADR-018.

- Default weight: **Regular**.
- Emphasis / active state: **Bold**.
- Duotone / Fill: reserved for illustrations, not UI icons.
- Import per-icon: `import { FileText } from '@phosphor-icons/react'`.
  Never `import * as Icons`.
- Icon size follows type it accompanies:
  - Next to `body-md` (14/22): 16px icon
  - Next to `body-lg` (16/26): 18px icon
  - Next to `heading-md`: 20px icon
- Stroke weight: use icon's own; do not restyle.
- Never combine Phosphor with another icon library — pick one.

---

## Component Composition

Components live in `packages/ui`. Rules:

- **Headless first**: wrap Radix Primitive; expose the same
  contract Radix does; add styling.
- **Compound components**: `<Dialog>`, `<Dialog.Trigger>`,
  `<Dialog.Content>` — not opaque props.
- **`asChild` prop**: accept it for interactive wrappers so styling
  composes without losing semantics.
- **No default `className` merging with `clsx` guesswork**. Use
  `tailwind-merge` explicitly.
- **Storybook**: every component has at minimum a `Default` and
  `Playground` story. Complex components have state-specific
  stories (`Loading`, `Error`, `Empty`).

Component naming: `PascalCase`. Compound parts as static
properties: `Dialog.Content`. Utility hooks: `useDialog`.

Detail: R605.

---

## Empty / Loading / Error States

Skipping these is the loudest tell of a rushed / generated UI.
Every page and every list has three deliberately designed states.

**Empty state**:
- One line of copy that's specific to the surface ("No contracts
  yet. Upload your first NDA to see it come alive.").
- One action (primary CTA).
- Optional: small illustration in monochrome (not emoji, not stock).
- Never: "No data" plus a shrugging emoji.

**Loading state**:
- Skeleton loaders **only** when server latency ≥200ms.
- Otherwise: content pop-in with `swift` easing.
- Never: full-page spinners for anything except cold-start.
- Streaming (LLM): show tokens as they arrive; use a subtle
  animated cursor to signal ongoing generation.

**Error state**:
- Specific: "Couldn't reach Qdrant. Retrying in 5s" — not
  "Something went wrong."
- Action: retry button + link to status page (or `mailto:` for MVP).
- Preserve state: do not clear the user's input on error.
- Log to backend for observability; also show a short error ID
  the user can quote to support.

Detail: R604.

---

## Density Philosophy

**Default is high-density**. B2B users have wide screens and want
information. A page with 15 rows of contract summaries and a
sidebar of filters is preferable to a page with 4 giant cards and
whitespace around each.

**Test**: on a 1440×900 laptop screen, a list view should show
≥15 items visible without scroll, or explain why not (e.g., a
detail-heavy view intentionally showing 6 with rich preview).

**Whitespace budget**: whitespace is for hierarchy and rhythm, not
for aesthetic. If you can add another useful column without
crowding, add it.

---

## Dark Mode

Not a color inversion — a first-class theme.

- Base neutral in dark: `sand-dark` (Radix). Warm, less contrast
  fatigue than pure grayscale.
- No pure black backgrounds; step 1 = `#171512` (approx), not
  `#000`.
- Text step 12 does not reach pure white; `#F5F1EB` (approx).
- Accent adjusts: `iris-dark` (slightly desaturated for reduced
  glare).
- Semantic colors adjust to remain WCAG AA at each step in dark
  context.

Toggle mechanism: `data-theme="light" | "dark"` on `<html>`, driven
by system preference by default with an override in user settings.
No FOUC — theme applied before first paint via inline script in
`_document.tsx`.

---

## Accessibility Baseline

Non-negotiable minimum:

- **Contrast**: WCAG AA (4.5:1 for text, 3:1 for large text and
  UI). Radix Colors' step-11 on step-1 already meets this by design.
- **Focus**: visible focus ring on every interactive element,
  minimum 2px, using step-7 of the accent color. Never `outline: none`
  without a replacement.
- **Keyboard**: every UI action is keyboard-reachable. Every dialog
  traps focus while open and returns focus on close.
- **ARIA**: Radix handles most of this; do not undo it.
- **Motion**: `prefers-reduced-motion` respected (see Motion section).
- **Screen reader**: form fields have labels; icons that convey
  meaning have `aria-label`; decorative icons `aria-hidden="true"`.
- **Language**: `<html lang>` reflects UI language (per ADR-025).

Testing: Lighthouse CI gate at ≥95 accessibility score (per ADR-020).
Manual audit of critical flows once per feature.

---

## Changelog

- **2026-07-21** — Initial version. Locked against ADR-015 through
  ADR-018 and ADR-025.
