# .harness/ — Harness Engineering for ContractIQ

This folder holds the engineered environment that guides AI agents
working on ContractIQ. It is designed to be **model-agnostic** —
the rules and prompts must produce reliable output even with
smaller models (DeepSeek, Haiku, Gemini Flash), not only frontier
models.

## Structure

.harness/
├── README.md # This file
├── rules/ # Numbered rules (R###)
├── prompts/ # Task templates (copy-paste into chat)
├── checklists/ # Pre-commit / pre-merge / release
└── examples/ # Good/bad example outputs

## Rule Numbering

Rules are numbered by category. Each rule has a stable ID (`R###`)
so it can be cited concisely in reviews and diffs.

| Range     | Category              |
| --------- | --------------------- |
| R000-R099 | Meta (how to use)     |
| R100-R199 | Coding standards      |
| R200-R299 | Commit & git          |
| R300-R399 | Testing               |
| R400-R499 | Security              |
| R500-R599 | AI integration        |
| R600-R699 | Design system         |
| R700-R799 | Reserved              |
| R800-R899 | Reserved              |
| R900-R999 | Role & mode (highest priority) |

Higher-numbered categories generally take precedence when they
conflict (R900 > R500 > R100). Individual rules can override this
by explicit "OVERRIDES" clauses.

## How To Use

- Every new task: agent reads `CLAUDE.md`, then the files it points
  to, then any rule the task touches.
- Human review: cite rule IDs when giving feedback ("violates R101",
  "see R204").
- Amending rules: never edit silently. Add an ADR to `DECISIONS.md`
  first, then update the rule in the same commit.

## Prompt Templates

`prompts/` contains ready-to-paste templates for common tasks:

- `_preamble.md` — MUST be conceptually loaded at task start
- `plan-feature.md` — turn a feature description into a task list
- `implement-feature.md` — propose diffs for a scoped task
- `review-diff.md` — review a proposed diff against rules
- `debug-error.md` — structured debugging flow
- `design-tool.md` — design an LLM tool schema
- `design-prompt.md` — design an LLM prompt

Copy the template into the chat, fill in the placeholders, send.

## Advisor-Only Mode

This project runs in **advisor-only mode**: the agent proposes,
the human executes. See `rules/R900-advisor-only-mode.md`.
