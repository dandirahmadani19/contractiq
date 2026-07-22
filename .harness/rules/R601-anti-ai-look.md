# R601 — Anti-AI-Look Rules

Rules to prevent the visual patterns that mark UI as "AI-generated".
Each rule is a specific ban with concrete alternatives.

## R601.1 — No Purple-Blue Gradients

**MUST NOT** use gradients of the form purple → blue, indigo → violet,
or any diagonal / conic gradient for backgrounds, buttons, text,
or borders.

### Violation
```tsx
<h1 className="bg-gradient-to-r from-purple-500 to-blue-500 bg-clip-text text-transparent" />
```

### Correct
```tsx
<h1 className="text-text-primary font-display" />
```

Rationale: this exact pattern is the most-copied "AI look" from
2024 SaaS landing pages.

## R601.2 — No Sparkle / AI Emojis in UI

**MUST NOT** use ✨ 🚀 ⚡ 🎯 🔥 in copy, buttons, or icons.
**MUST NOT** use phrases: "AI-powered ✨", "Powered by AI",
"Transform your workflow", "Revolutionize".

### Violation
```tsx
<Button>Try our AI ✨</Button>
```

### Correct
```tsx
<Button>Analyze contract</Button>
```

Rationale: sparkle emojis in AI product copy read as marketing
noise. Specificity of function > adjectives about AI.

## R601.3 — No Uniform `rounded-2xl` or Larger

**MUST NOT** apply `rounded-2xl`, `rounded-3xl`, or `rounded-full`
uniformly across a UI. Only `pill` badges and avatars may be fully
rounded.

### Violation
```tsx
<Card className="rounded-2xl">...</Card>
<Button className="rounded-2xl">...</Button>
<Input className="rounded-2xl">...</Input>
```

### Correct
```tsx
<Card className="rounded-md">...</Card>       // 6px
<Button className="rounded-sm">...</Button>   // 4px
<Input className="rounded-sm">...</Input>     // 4px
<Badge className="rounded-pill">...</Badge>   // 999px OK for pill
```

Rationale: uniform large radius creates the "Dribbble mockup"
look. Modest, differentiated radius reads as considered.

## R601.4 — No Gradient Text on Headings

**MUST NOT** use gradient-clipped text for headings.

### Violation
```tsx
<h1 className="bg-gradient-to-r from-iris-9 to-iris-11 bg-clip-text text-transparent">
  ContractIQ
</h1>
```

### Correct
```tsx
<h1 className="text-text-primary font-display">
  ContractIQ
</h1>
```

Rationale: mainstream since Vercel's landing page ~2022; now a
tell. Serif display type carries identity without gimmicks.

## R601.5 — No Glassmorphism Without Reason

**MUST NOT** use `backdrop-blur` on decorative surfaces.
**MAY** use `backdrop-blur-sm` for functional overlays where content
behind must remain hint-visible (e.g., dialog scrim, top nav over
scrolled content).

### Violation (decorative)
```tsx
<Card className="backdrop-blur-xl bg-white/30 border border-white/20">
```

### Correct (functional)
```tsx
<DialogOverlay className="backdrop-blur-sm bg-black/40" />
```

## R601.6 — No Universal Hover Scale

**MUST NOT** apply `hover:scale-*` to elements without a specific
interaction reason.

### Violation
```tsx
<Card className="hover:scale-105 transition" />  // every card scales
```

### Correct
```tsx
<Card className="hover:bg-bg-elementHover transition-colors duration-fast" />
```

Rationale: universal hover scale is over-used in AI-generated
Framer prototypes. Prefer background/border/color feedback.

## R601.7 — No Emoji as Icons

**MUST NOT** use emoji in place of icons in the UI.
**MUST**: use Phosphor Icons per ADR-018.

### Violation
```tsx
<button>📄 Upload</button>
```

### Correct
```tsx
import { FileText } from '@phosphor-icons/react';
<button><FileText size={16} /> Upload</button>
```

Exception: user-generated content and search results may contain
emoji; the UI itself must not.

## R601.8 — No Symmetric Hero + 3 Cards + CTA Layout

**MUST NOT** structure marketing / landing pages as: full-width hero
→ 3 equal feature cards → centered CTA. **MUST**: prefer asymmetric,
information-dense layouts (see R605 for composition patterns).

Rationale: this exact template is the "landing page starter"
pattern shipped with every AI website builder. Recognizable
= generic.

## R601.9 — No Default Tailwind Slate / Zinc

**MUST NOT** use `slate-*` or `zinc-*` classes. **MUST**: use
Radix `sand` scale via semantic tokens (R600.2).

### Violation
```tsx
<div className="bg-slate-50 text-slate-900" />
```

### Correct
```tsx
<div className="bg-app text-text-primary" />
```

## R601.10 — No Lorem Ipsum, No Generic Mock Data

**MUST NOT** use lorem ipsum in any state (Storybook, marketing,
empty states, tests).
**MUST**: use real-shaped mock data — plausible contract titles,
plausible party names, plausible dates.

### Violation
```
Product Foo
Lorem ipsum dolor sit amet
Created: 2024-01-01
```

### Correct
```
Vendor Master Services Agreement — Acme × Widget Co
NDA — bilateral, 3-year term
Signed: March 14, 2026
```

Rationale: mock data is design decision. Realistic data reveals
layout issues that lorem hides (long titles wrap, date formats
matter).

## R601.11 — No Unbacked Numeric Claims

**MUST NOT** put numeric performance claims in UI or marketing
("10x faster", "99% accurate", "3 seconds") unless the number is
sourced from a measurement recorded in `learning_docs/measurements/`.

### Violation
```tsx
<p>Analyze contracts 10x faster.</p>
```

### Correct
```tsx
<p>Review a 30-page NDA in under 8 seconds.</p>
{/* This number sourced from learning_docs/measurements/2026-08-01-ingestion-benchmark.md */}
```

## R601.12 — No Fade-and-Scale Dialog Enters

**MUST NOT** open dialogs with combined fade + scale + Y motion —
the "Dribbble modal" pattern.
**MUST**: use single-property motion (opacity + Y translate ≤8px)
with `swift` easing.

## R601.13 — No Contentless Loading Spinners

**MUST NOT** show a bare spinner without indicating what is loading.
**MUST**: either use skeleton (R604) or provide context text
("Analyzing 12 clauses...").

## R601.14 — No Testimonial Section with Stock Photos

**MUST NOT** include testimonial sections with stock avatars,
fabricated quotes, or invented person names in the MVP landing page.
**MAY** include real testimonials from real named users after v0.1.0
launch.
