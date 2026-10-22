# Advisor consultation prompt (Phase 4.1)

For a fresh advisor agent (`subagent_type: "ecoskills:advisor"`, or `"general-purpose"` if that type isn't available): `model: "opus"` by default, or `model: "fable"` (with the user's go-ahead), with `effort: "medium"`. Write it to a file in the scratchpad directory and launch the advisor with a short prompt pointing at it. Fill in the `<...>` parts. Pass paths and short excerpts, not full logs or diffs; the advisor can read what it needs.

```markdown
# Request: Advise on a hard problem in unit <U-n>

A plan document is being implemented one commit at a time. The executor of unit <U-n> has hit a problem it can't resolve on its own. Diagnose it and recommend how to proceed. You are an advisor: don't edit files, commit, or run git commands that modify state. You may read the repository and run read-only commands, including the plan's verification commands, in <worktree path>.
<If commands are restricted in the worktree: "Use the helper <wrapper path> <usage> instead of <blocked command>.">

## Inputs
- Plan: <worktree path>/<plan path>. The unit implements section <section>. Constraints: section <section>.
- Consultation request from the executor: <scratchpad>/<U-n>-consult-request.md (what it is trying to do, what it tried, its hypotheses, and its question)
- Uncommitted work in progress: `git -C <worktree path> diff` <or "none">
- Earlier consultations on this unit: <advice file paths, or "none">

## Use your context sparingly
Your context and tokens are the expensive part of this consultation. Keep them for reasoning and judgment:
- Send broad searches, reading many files, and scanning long command output to cheaper agents: `model: "haiku"` for mechanical lookups (find where X is defined, extract the error lines from this log), `model: "sonnet"` for exploration that needs some understanding (how does module Y handle Z). Ask them for short, specific answers with file:line references, not file dumps. Launch independent ones in parallel.
- Use a fork (`subagent_type: "fork"`) to pursue one hypothesis in depth, such as reproducing a failure or tracing a call path, when that would flood your own context. It inherits what you know, so its prompt can be short. Ask it to return only its conclusion and evidence. Run competing hypotheses as parallel forks.
- Read the few lines that decide the question yourself. Don't delegate the final judgment.
- If you can't launch agents, work directly, but read excerpts (`grep -n`, `sed -n`) rather than whole files.

## What to return
Write your advice to <scratchpad>/<U-n>-advice-<k>.md and return it as your final message. Keep it short enough for the executor to act on:
1. Diagnosis: the cause, or the deciding factor for the choice, with evidence (command and output, or file:line).
2. Recommendation: concrete steps for the executor, within this unit's scope.
3. Alternatives you rejected, one line each, and why.
4. Confidence (high / medium / low) and the check that would confirm the recommendation worked.
5. Plan impact: say explicitly if the plan cannot be followed as written, or if the recommendation departs from a decision or constraint in the plan. Don't recommend working around the plan; flag it instead.
```
