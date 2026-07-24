# progress.md — Current State & Session Log

**Purpose**: Single source of truth untuk "di mana kita sekarang".
Setiap sesi coding dimulai dengan baca file ini. Setiap sesi
selesai dengan update file ini.

## Current Phase

**Foundation (Phase 0-3): DONE.**

- Phase 0 (Bootstrap): DONE 2026-07-21
- Phase 1 (Lock Decisions): DONE 2026-07-21
- Phase 2 (Rules Complete): DONE 2026-07-22
- Phase 3 (Learning Docs Foundation): DONE 2026-07-22

**Next Phase: Coding** — starting from `T-001` per `MVP.md`.

## Current Task

**Next up: T-003 — Root TypeScript config + strict flags.**

- Dependencies: T-002 ✓
- Est: S (≤ 1h focused)
- Files: `tsconfig.base.json`, `tsconfig.json`
- Full spec: see `MVP.md#T-003` for verification steps

## Session Log

Session log format:
- Date + brief goal
- Tasks touched (mark ✓ done, ~ in-progress, ! blocked)
- Learnings or decisions worth noting
- Next action

Newest entries at top.

### 2026-07-24 — T-002 done

- **T-002 ✓** — Root `biome.json`, `.editorconfig`, `.gitattributes`
  created (R100.2, R100.5, R100.6 seeded at root; full CI-gate
  rule set deferred to `packages/config/biome/base.json` in T-004).
  `@biomejs/biome@1.9.4` added as workspace devDependency.
  Verified: `pnpm biome check .` passes (zero files, exit 0).
- Confirmed `biome.json` sits in R900.9 (runtime/config zone per
  ADR-027) — proposed via R900.1–.4, not auto-edited.
- **Next**: T-003 (root `tsconfig.base.json` + `tsconfig.json`,
  per ADR-003 strict flags).

### 2026-07-22 — T-001 done + R900 amended via ADR-027

- **T-001 ✓** — pnpm workspaces + Turborepo initialized.
  Verified: `pnpm --version` = 9.15.0, `pnpm turbo --version` =
  2.10.6, `pnpm ls -r --depth -1` shows root package
  `contractiq@0.0.0` (workspace members not yet created, expected).
- **ADR-027** — R900 amended: documentation zone auto-edit
  permitted (R900.8), runtime zone advisor-only remains
  (R900.9), constitutional edits require propose step (R900.10).
  Rationale: friction reduction for doc churn without weakening
  runtime safety.
- **Transition point**: this is the last batch executed under
  OLD R900. From next task (T-002) onward, new convention
  active — Sonnet may auto-edit doc zone.
- **Next**: T-002 (Biome + editorconfig + gitattributes).

### 2026-07-22 — Foundation complete, ready for coding

- All rules locked (R000-R900, 197 sub-rules).
- ADR-026 supersedes ADR-012 (free-tier first LLM strategy).
- Learning docs foundation seeded (5 files).
- **Next**: T-001 in fresh Claude Code session (Sonnet).

## How To Update This File

At end of each session:

1. Update **Current Task** if task changed.
2. Add new session entry at top of **Session Log**.
3. Keep session entries brief (5-10 lines each).
4. Commit as part of the task's commit, or standalone:
   `docs: update progress after T-###` or
   `docs(harness): log session <YYYY-MM-DD>`.

## How To Read This File (for Claude at session start)

1. Read **Current Phase** — where we are in the roadmap.
2. Read **Current Task** — what to work on next.
3. Read the latest 1-2 **Session Log** entries — recent context.
4. Cross-reference `MVP.md` for the full task spec.
5. Cross-reference relevant `ADR-###` and `R###` per the task's
   citations.

Do NOT read the entire Session Log unless investigating history.
Recent entries + current task = enough context for most sessions.
