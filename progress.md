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

**Next up: T-001 — Init pnpm workspaces + Turborepo.**

- Dependencies: none
- Est: S (≤ 1h focused)
- Files: `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `.npmrc`
- Full spec: see `MVP.md#T-001` for verification steps

## Session Log

Session log format:
- Date + brief goal
- Tasks touched (mark ✓ done, ~ in-progress, ! blocked)
- Learnings or decisions worth noting
- Next action

Newest entries at top.

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
