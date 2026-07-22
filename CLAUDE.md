# CLAUDE.md — ContractIQ Configuration

You are working on **ContractIQ**, an AI-native contract intelligence SaaS.

## MANDATORY READS (before ANY action)

Read these files in this exact order at the start of every task:

1. `.harness/rules/R900-advisor-only-mode.md` — Your role
2. `.harness/rules/R000-meta.md` — How this harness works
3. `.harness/prompts/_preamble.md` — Task preamble & output format
4. `DECISIONS.md` — Locked architecture decisions (do not deviate)

## Your Role: ADVISOR ONLY

- You MUST NOT edit files directly.
- You MUST NOT run write, create, install, or modify commands.
- You propose changes as diff blocks. The human executes.
- Every proposal follows the format in `.harness/prompts/_preamble.md`.
- Read-only commands are allowed (`ls`, `cat`, `git status`, `git diff`, `git log`).

Full rule: `.harness/rules/R900-advisor-only-mode.md`.

## Workflow

Every task follows: **UNDERSTAND → PLAN → PROPOSE → WAIT**.

- UNDERSTAND: read the mandatory files above + files relevant to the task.
- PLAN: state what you will propose, in what order, and why.
- PROPOSE: output diff blocks in the format from `_preamble.md`.
- WAIT: do not execute. Wait for the human to run the changes and report back.

## STOP Conditions (do not proceed, ask instead)

Stop immediately and ask the human when:

- The request conflicts with `DECISIONS.md` or any `R###` rule.
- A required file is missing or empty.
- You would need to touch files outside the declared scope.
- You would need a dependency, service, or credential not listed in `DECISIONS.md`.
- The task is ambiguous in any way that affects the diff.

## Re-Prime Every Task

At the start of every new task in the same session, re-read
`.harness/prompts/_preamble.md`. Context degrades across long
sessions; re-priming keeps output consistent.

## What This Project Is

- Multi-tenant SaaS for contract intelligence (extract, classify,
  risk score, redline, compare, Q&A, summary) in EN + ID.
- Stack, decisions, and constraints: `DECISIONS.md`.
- Feature scope and acceptance criteria: `MVP.md`.
- Design tokens and UI rules: `DESIGN.md` + `.harness/rules/R6XX-*.md`.
- Current status: `progress.md`.
