# R605 — Component Composition

## R605.1 — Foundation: Radix Primitives + Tailwind

**MUST**: build interactive components as thin wrappers around
Radix UI Primitives.
**MUST NOT**: use shadcn/ui, MUI, Chakra, Mantine, Ant Design, or
any styled component library.

Rationale: ADR-015. Owning the styling layer is the point.

## R605.2 — Compound Component Pattern

**MUST**: expose the same compound API Radix exposes.

### Example
```tsx
// packages/ui/src/Dialog/index.tsx
import * as RadixDialog from '@radix-ui/react-dialog';

export const Dialog = RadixDialog.Root;
Dialog.Trigger = RadixDialog.Trigger;
Dialog.Content = styled(RadixDialog.Content, ...);
Dialog.Title = styled(RadixDialog.Title, ...);
Dialog.Description = styled(RadixDialog.Description, ...);
Dialog.Close = RadixDialog.Close;
```

**MUST NOT**: bury Radix behind opaque props like
`<Dialog title="..." description="..." />`.

## R605.3 — `asChild` Support

Interactive wrappers MUST accept `asChild` when Radix does. This
lets consumers compose without losing semantics or focus behavior.

## R605.4 — className Merging

**MUST**: use `tailwind-merge` (via `cn()` helper) to combine
classes.
**MUST NOT**: use `clsx` alone (produces class conflicts silently).

```tsx
// packages/ui/src/utils/cn.ts
import { twMerge } from 'tailwind-merge';
import { clsx, type ClassValue } from 'clsx';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## R605.5 — Variants via CVA

**MUST**: use `class-variance-authority` for component variants
(button size, badge tone, etc.).
**MUST NOT**: use ternary chains inline for variant logic.

## R605.6 — Naming

- Components: `PascalCase`.
- Compound parts: static properties (`Dialog.Content`).
- Hooks: `useX` (`useDialog`).
- Utility helpers: `camelCase` (`getContrastText`).

## R605.7 — Storybook Contract

Every component in `packages/ui` MUST have:

- `Default` story showing typical use.
- `Playground` story with all props controllable via Storybook
  controls.
- Stories for each variant (e.g., `Sizes`, `Tones`).
- State stories when applicable (see R604.6).
- A11y check via `@storybook/addon-a11y` runs on every story.

**MUST NOT** merge a new component to `main` without a story.

## R605.8 — Accessibility Preservation

Radix ships with correct ARIA and keyboard handling.

**MUST NOT** override:
- Focus management.
- Keyboard event handlers.
- ARIA attributes.
Without explicit review noted in the component's Storybook description.

If you must override for a genuine reason, log it as `// waives: R605.8` with rationale inline.

## R605.9 — Testing

Every component MUST have:

- Render smoke test (imports and renders without crashing).
- Interaction test for stateful components (Vitest + Testing Library).
- Visual regression (Chromatic) for critical components (button,
  input, dialog, dropdown, table).

## R605.10 — Directory Convention

```
packages/ui/src/
├── <ComponentName>/
│   ├── index.tsx          # public export
│   ├── styles.ts          # variants (CVA)
│   ├── types.ts           # props types
│   ├── <ComponentName>.stories.tsx
│   └── <ComponentName>.test.tsx
├── tokens/                # design tokens (R600)
├── utils/                 # cn, hooks
└── index.ts               # barrel export
```
