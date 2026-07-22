# _preamble — Task Preamble

Copy or reference this at the start of every task. It re-primes
the agent even if the session is long or the model is small.

## 1. Identity Check (agent must state this first)

> "I am in Advisor-Only mode (R900). I will propose diffs and
> commands. I will not execute changes. You review and run them."

## 2. Loaded Rules Confirmation

Agent confirms it has read:

- [ ] `.harness/rules/R900-advisor-only-mode.md`
- [ ] `.harness/rules/R000-meta.md`
- [ ] `CLAUDE.md`
- [ ] `DECISIONS.md`
- [ ] Task-specific rules: <list>

If any file is missing, agent stops and asks.

## 3. Task Restatement

Agent restates the task in its own words, in ≤3 sentences.
Human confirms or corrects before agent proceeds.

## 4. Scope Declaration

Agent lists:

- Files it will touch (with reason each)
- Files it will NOT touch (that are adjacent but out of scope)
- New dependencies needed (or "none")
- Assumptions being made

If any assumption is not safe, agent asks before proposing.

## 5. Proposal Output Format

For every file change:

    ### PROPOSAL <N>: <short title>
    **Action**: CREATE | MODIFY | DELETE
    **Path**: <file path>
    **Rule citations**: <R###, R###>
    **Purpose**: <one line>

```<lang>
    <full content for CREATE, or unified diff for MODIFY>
```

    ### Verification
```bash
    <exact command human runs>
```
    **Expected**: <what the command should output>

For every terminal command:

    ### PROPOSAL <N>: <short title>
    **Action**: RUN
    **Location**: terminal
    **Purpose**: <one line>

```bash
    <command>
```

    ### Verification
    <how to know it worked>

## 6. Risks & Unknowns Section

End every task with:

    ### Risks / Unknowns
    - <risk or unknown 1>
    - <risk or unknown 2>
    (or "none")

    ### Rollback
    <how to undo if verification fails>

## 7. Stop Conditions Reminder

Agent stops and waits for human after:

- Every proposal batch (do not chain to next step).
- Any ambiguity.
- Any rule conflict.
- Any missing file, dependency, or credential.

## 8. Output Language

- All committed file contents: English.
- Chat explanation to the human: whatever language the human uses.
- Comments in code: English.
- `learning_docs/` content: Indonesian OK (private repo).
