# R000 — Meta Rules

These rules govern how all other rules are interpreted and applied.

## R000.1 — Rules Are Read At Task Start

Every task starts by reading `CLAUDE.md`, which points to the
mandatory files. Do not rely on remembered rules from earlier in
the session. Re-read.

## R000.2 — Cite Rule IDs In Proposals

Every diff or proposal must cite the rule IDs it relies on.

**Good**:
> Chose `rounded-md` (per R601.3, avoid `rounded-2xl` everywhere).

**Bad**:
> Chose `rounded-md` because it looks better.

## R000.3 — Ambiguity → Ask, Do Not Improvise

If a rule is ambiguous or two rules conflict without an explicit
override, stop and ask the human. Do not pick an interpretation
and proceed.

## R000.4 — Higher Category Wins Ties

When rules from different categories give conflicting guidance
and no explicit override exists, the higher-numbered category
wins (R900 > R500 > R100). Individual rules can override this
with an "OVERRIDES: R###" clause.

## R000.5 — Rule Changes Require an ADR

Never edit a rule file silently. First add an ADR entry to
`DECISIONS.md` describing:

- Which rule changes
- Why (what problem the change solves)
- What the old and new behavior are

Then update the rule file in the same commit.

## R000.6 — Rules Are For The Agent AND The Human

Rules are self-descriptive. If a human ignores a rule during
review, the agent still applies the rule. If the human wants an
exception, they add a `// waives: R###` comment inline explaining
why.

## R000.7 — Model-Agnostic Wording

Rules must be written in short, explicit sentences that a smaller
model can follow without inference. Prefer:

- MUST / MUST NOT / SHOULD / SHOULD NOT / MAY
- Concrete examples (both positive and negative)
- Numbered sub-rules for granular citation (R101.2, R101.3)

Avoid:

- Vague verbs ("consider", "try to", "typically")
- Rules that require reading between the lines
- Rules that only make sense with world knowledge outside the repo
