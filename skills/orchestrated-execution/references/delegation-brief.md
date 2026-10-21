# Delegation brief template (Phase 4.1)

Write this to a file in the scratchpad directory and launch the Sonnet agent with a short prompt pointing at it.

```markdown
# Task: implement unit <U-n> (<plan task ID>: <title>)

## Where
- Worktree: <absolute worktree path>. Work only here. Keep each Bash command simple (no `cd`, no chained commands) and run it from this directory.
- <If commands are restricted in the worktree: "Use the helper <wrapper path> <usage> instead of <blocked command>.">
- Comments you write must not reference the plan document; it is deleted before integration.
- Plan: <worktree path>/<plan path>. Read sections <the unit's task section>, <decisions it cites>, and <constraints section>. The plan is the specification.

## What
<The unit's purpose and contents in 2-5 lines, quoting the exact snippets or commands from the plan when it gives them.>
This summary is a guide. Where it and the plan sections listed above differ, or where it leaves something out, the plan wins. Implement every item those sections specify for this unit.

Decisions made with the user that refine the plan: <list, or "none beyond what the plan records">

## Rules
- Implement only this unit. Don't start the next one, and don't "improve" things outside it.
- Follow the plan's constraints, in particular: <the 2-4 constraints most relevant to this unit>.
- Don't modify existing tests to make them pass. If a completion criterion can't be met as written, stop and report it. Don't work around it.
- Run every completion criterion below and capture the actual output.
- Make exactly one commit: <message convention, plus attribution lines if any are in effect>.

## Completion criteria
<copied from the plan, exact commands and expected results>

## Report (final message; also write it to <scratchpad>/<U-n>-report.md)
1. Commit SHA and the files changed
2. Each completion criterion: command, actual output, pass/fail
3. Any deviation from the plan, and why
4. Anything you noticed that may affect later units
```
