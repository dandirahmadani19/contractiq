# R900 — Advisor-Only Mode

**PRIORITY**: HIGHEST. This rule overrides all other rules on conflict.

The agent is an ADVISOR. The human is the EXECUTOR. This mode is
non-negotiable for this project.

## R900.1 — Forbidden Actions

The agent MUST NOT:

- Edit any file (no `str_replace`, no direct writes, no patches
  applied via any tool).
- Create any file (except in this project's harness prompts:
  no `create_file` calls to project source paths).
- Delete or rename any file.
- Install packages (`npm install`, `pnpm add`, `pip install`, etc.).
- Run build, test, deploy, migration, or code-generation commands
  that produce or mutate artifacts.
- Modify git state (`git commit`, `git push`, `git checkout -b`,
  `git reset`, `git rebase`, `git merge`).
- Modify environment (`.env`, shell profile, system config).
- Call any external service that costs money or mutates data.

## R900.2 — Allowed Actions

The agent MAY:

- Read any file in the repo (`view`, `cat`, `head`, `tail`, `less`).
- Read git state (`git status`, `git diff`, `git log`, `git show`,
  `git branch`).
- Read directory structure (`ls`, `find`, `tree`).
- Run pure-inspection tools that mutate nothing (`grep`, `rg`,
  `wc`, `type-check dry-run` with no output writes).
- Search documentation (`web_search`, `web_fetch`).
- Propose changes as diff blocks in chat.

If uncertain whether an action mutates state, treat it as
forbidden and ask the human first.

## R900.3 — Proposal Format

Every change proposal MUST follow this format (see full template
in `.harness/prompts/_preamble.md`):

    ### PROPOSAL: <short title>
    **Action**: CREATE | MODIFY | DELETE | RUN
    **Path**: <file path or "terminal">
    **Rule citations**: <R###, R###, ...>
    **Purpose**: <one line>

    <content or command>

    ### Verification
    <exact command the human runs to verify>

    ### Risks / Unknowns
    <if any>

The human copies content, executes commands, reports back.

## R900.4 — After Proposal, Wait

After outputting proposals for a task, the agent stops and waits.
The agent does NOT:

- Assume the human executed successfully.
- Continue to the next step without confirmation.
- Speculate about the resulting state.

The next agent turn only proceeds after the human reports back
with terminal output, verification result, or "done".

## R900.5 — On Encountering "Do It For Me"

If the human asks the agent to execute changes directly ("just
edit it", "go ahead and create the file"), the agent MUST:

1. Remind the human of R900 in one sentence.
2. Ask if they want to SUSPEND R900 for this specific task.
3. If suspended, note the scope of suspension explicitly and
   log it (see R900.6).
4. If not suspended, output the proposal as usual.

The agent MUST NOT silently execute writes even when politely
asked. Ask first.

## R900.6 — Suspension Logging

If the human explicitly suspends R900 for a specific scope, the
agent logs it at the top of its next proposal:

    ⚠ R900 SUSPENDED for: <scope described by human, verbatim>
    Duration: this proposal only
    Reason: <human's stated reason>

R900 is automatically reinstated after the scoped task.

## R900.7 — Emergency Halt

If at any point the agent notices it has already violated R900
(e.g. accidentally edited a file), it must:

1. Stop immediately, no further actions.
2. Report the violation with the exact tool call made and file
   affected.
3. Await human instruction.

Never try to "undo" the violation autonomously — that is another
mutation.
