# R602 — Typography

## R602.1 — Three Fonts, Three Roles

Per ADR-017 and DESIGN.md:

| Role                | Font              | Tailwind class    |
| ------------------- | ----------------- | ----------------- |
| Display / heading   | Instrument Serif  | `font-display`    |
| Body / UI           | Inter             | `font-body`       |
| Data / mono         | Geist Mono        | `font-mono`       |

**MUST NOT**: use any other font family.
**MUST NOT**: use Google Fonts CDN in production — self-host via
`@next/font` local.

## R602.2 — Instrument Serif Usage

**MAY** be used for:
- `display-2xl`, `display-xl`, `display-lg` (hero, page titles,
  section leads).
- `heading-lg` when the surface is editorial (landing pages, docs
  home).

**MUST NOT** be used for:
- Body text.
- Buttons, form labels, nav items.
- All-caps content.
- Weights other than Regular and Italic Regular.

Italic Regular replaces bold as emphasis.

## R602.3 — Inter Usage

**MUST** be used for:
- All body prose.
- All UI controls.
- Headings `heading-md` and smaller.

**MUST**: enable OpenType feature `ss01` (single-story `a`).
**MAY**: enable `cv11` for slashed zero in mixed-mono contexts.
**MUST NOT**: use weights ≥700; use 400/500/600 only.

## R602.4 — Geist Mono Usage

**MUST** be used for:
- Contract IDs, chunk IDs, timestamps in ISO format.
- Numeric data in dense tables (columns of counts, prices, scores).
- Keyboard shortcuts (`⌘K`, `Enter`).
- Code blocks in docs.

**MUST NOT** be used for:
- Prose.
- Interactive labels.

## R602.5 — Typography Scale

Consume via Tailwind classes; do not use raw font-size / line-height
in components.

| Class              | Font/size/leading            |
| ------------------ | ---------------------------- |
| `text-display-2xl` | Serif 56/60                  |
| `text-display-xl`  | Serif 40/46                  |
| `text-display-lg`  | Serif 32/40                  |
| `text-heading-lg`  | Inter Semibold 24/32         |
| `text-heading-md`  | Inter Semibold 18/26         |
| `text-heading-sm`  | Inter Semibold 16/24         |
| `text-body-lg`     | Inter Regular 16/26          |
| `text-body-md`     | Inter Regular 14/22          |
| `text-body-sm`     | Inter Regular 13/20          |
| `text-caption`     | Inter Medium 12/18           |
| `text-mono-md`     | Geist Mono Regular 13/20     |
| `text-mono-sm`     | Geist Mono Regular 12/18     |

## R602.6 — Line Length

**SHOULD**: cap prose at 65–75 characters per line
(`max-w-prose` ≈ 65ch).
**MUST NOT**: leave prose full-width on wide layouts.

## R602.7 — Hierarchy Discipline

Skipping heading levels is prohibited. If a page has an `H1`, the
next heading is `H2`, not `H3`. Use `text-*` classes to control
appearance without violating semantic order.
