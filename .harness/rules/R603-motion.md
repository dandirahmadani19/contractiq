# R603 — Motion

## R603.1 — Motion Must Communicate

**MUST**: every motion in the UI communicates a state change
(entering, leaving, focusing, transitioning between states).
**MUST NOT**: use motion for decoration.

Test: "If I remove this motion, does the user lose information?"
If no, remove it.

## R603.2 — Duration Bands

Use tokens from R600.6. Bands:

| Interaction              | Duration  |
| ------------------------ | --------- |
| Focus, hover feedback    | `fast` (120ms)   |
| Dropdown, tooltip open   | `base` (180ms)   |
| Dialog, drawer open      | `slow` (240ms)   |
| Cross-page navigation    | `slower` (320ms) |
| Focus ring appearance    | `instant` (0ms)  |

**MUST NOT**: exceed `slower` (320ms) for any single motion. If a
transition needs more time, it's actually multiple transitions —
break it up.

## R603.3 — Easing

- Default: `swift` (`cubic-bezier(0.32, 0.72, 0, 1)`).
- Larger movement / non-UI (illustration): `smooth`.
- Continuous (progress, streaming, spinners): `linear`.

**MUST NOT**: use browser default `ease` or `ease-in-out`.

## R603.4 — Property Discipline

**MUST**: animate no more than 2 properties per motion.
**SHOULD**: prefer `opacity` and `transform` (compositor-cheap).
**MUST NOT**: animate `width`, `height`, `top`, `left`, `margin` —
prefer `transform` alternatives.

## R603.5 — Reduced Motion

**MUST**: honor `prefers-reduced-motion: reduce`. When set:
- Cross-fade instead of translate.
- Duration ≤ `fast` (120ms).
- No auto-playing motion (streaming cursor still allowed as
  functional signal).

Implementation: use motion library's reduced-motion hook, or CSS
`@media (prefers-reduced-motion: reduce)`.

## R603.6 — Page Transitions

**SHOULD**: use View Transitions API for same-app page navigation.
Persistent elements (sidebar nav, header) share their transition
name across pages.

## R603.7 — Streaming Motion

For LLM streaming (Q&A tokens):
- Tokens append with no animation per-token (motion would fight
  reading pace).
- A subtle animated cursor at the write head signals ongoing
  generation.
- On stream complete, cursor fades out over `base` (180ms).

## R603.8 — Skeleton Motion

Skeleton loaders use a horizontal shimmer, `linear`, 1.4s loop.
**MUST NOT**: use pulsing (fade in/out) — reads as spinner-in-place.

## R603.9 — No Hover Scale by Default

Restate R601.6 here for enforceability: **MUST NOT** apply
`hover:scale-*` as default hover feedback. Prefer background /
border color transition.
