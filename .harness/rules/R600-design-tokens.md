# R600 — Design Tokens

Every UI value MUST come from a named token defined here. No
arbitrary hex, pixel, or duration values in components.

## R600.1 — Token Location

All tokens live in `packages/ui/src/tokens/`. They are exported as
typed const objects and re-exported as CSS variables and Tailwind
theme extensions.

**MUST**: import tokens from `@contractiq/ui/tokens`.
**MUST NOT**: hardcode values in components.

### Example (correct)
```tsx
import { spacing } from '@contractiq/ui/tokens';
<div className="p-md gap-sm" />
```

### Example (violation)
```tsx
<div className="p-[12px] gap-[8px]" />  // ❌ R600.1
```

## R600.2 — Color Tokens

Palette source: Radix Colors 12-step scales. Semantic mapping only
(never reference step numbers in components).

**MUST**: reference semantic tokens (`bg-app`, `bg-subtle`,
`text-primary`, `border-default`).

**MUST NOT**: reference step numbers or hex values.

Semantic mapping (`packages/ui/src/tokens/color.ts`):

```typescript
export const color = {
  // Surface
  bg: {
    app: 'sand.1',
    subtle: 'sand.2',
    element: 'sand.3',
    elementHover: 'sand.4',
    elementActive: 'sand.5',
  },
  // Border
  border: {
    subtle: 'sand.6',
    default: 'sand.7',
    hover: 'sand.8',
  },
  // Text
  text: {
    primary: 'sand.12',
    secondary: 'sand.11',
    disabled: 'sand.10',
    inverse: 'sand.1',
  },
  // Accent (iris)
  accent: {
    bg: 'iris.3',
    bgHover: 'iris.4',
    border: 'iris.7',
    solid: 'iris.9',
    solidHover: 'iris.10',
    text: 'iris.11',
    textStrong: 'iris.12',
    focusRing: 'iris.7',
  },
  // Semantic (risk & feedback)
  semantic: {
    riskLow: 'green.11',       // low risk = good
    riskMedium: 'amber.11',
    riskHigh: 'crimson.11',
    riskLowBg: 'green.3',
    riskMediumBg: 'amber.3',
    riskHighBg: 'crimson.3',
    error: 'red.11',
    errorBg: 'red.3',
    success: 'green.11',
    successBg: 'green.3',
  },
} as const;
```

## R600.3 — Spacing Tokens

4pt grid, no exceptions.

```typescript
export const spacing = {
  xs:  4,
  sm:  8,
  md:  12,
  lg:  16,
  xl:  24,
  '2xl': 32,
  '3xl': 48,
  '4xl': 64,
  '5xl': 96,
} as const;
```

**MUST**: use spacing tokens for `padding`, `margin`, `gap`.
**MUST NOT**: use values off-grid (e.g., 10px, 14px, 20px).

## R600.4 — Radius Tokens

```typescript
export const radius = {
  none: 0,
  sm:   4,   // inputs, small buttons
  md:   6,   // cards, panels
  lg:   8,   // large surfaces, dialogs
  pill: 999, // badges, tags
} as const;
```

**MUST NOT**: use `rounded-2xl`, `rounded-3xl`, or `rounded-full` on
non-badge surfaces. See R601.6.

## R600.5 — Elevation Tokens

Shadows are minimal — density UI doesn't lean on shadow.

```typescript
export const shadow = {
  none: 'none',
  sm:   '0 1px 2px 0 rgba(0,0,0,0.04)',
  md:   '0 2px 6px 0 rgba(0,0,0,0.06), 0 1px 2px 0 rgba(0,0,0,0.04)',
  lg:   '0 8px 24px -4px rgba(0,0,0,0.10), 0 2px 6px 0 rgba(0,0,0,0.04)',
} as const;
```

**MUST NOT**: use `shadow-2xl` or any bloom effect.
**MUST NOT**: use colored shadows (e.g., glow effects) except for
focus rings.

## R600.6 — Motion Tokens

```typescript
export const motion = {
  duration: {
    instant: 0,
    fast:    120,
    base:    180,
    slow:    240,
    slower:  320,
  },
  easing: {
    swift:  'cubic-bezier(0.32, 0.72, 0, 1)',
    smooth: 'cubic-bezier(0.4, 0, 0.2, 1)',
    linear: 'linear',
  },
} as const;
```

**MUST**: use motion tokens in `transition` and `animate` props.
**MUST NOT**: use ad-hoc durations like `300ms`, `500ms`.

## R600.7 — Z-Index Tokens

Named scale to prevent stacking-context bugs.

```typescript
export const zIndex = {
  base:      0,
  dropdown:  10,
  sticky:    20,
  overlay:   30,
  dialog:    40,
  popover:   50,
  tooltip:   60,
  toast:     70,
} as const;
```

**MUST NOT**: use raw `z-` values in components.

## R600.8 — Typography Tokens

See R602 for full scale definition. Consumed via
`@contractiq/ui/tokens/typography` and applied as Tailwind
classes (`text-body-md`, `text-heading-lg`, etc.).

## R600.9 — Dark Mode Semantics

Every semantic token above resolves per `data-theme` attribute.
`data-theme="light"` maps to Radix light scale;
`data-theme="dark"` maps to Radix dark scale of the same hue.

**MUST**: apply theme via `data-theme` attribute on `<html>`.
**MUST NOT**: use Tailwind `dark:` class variant (we use attribute-based).
**MUST**: test every component in both themes in Storybook.
