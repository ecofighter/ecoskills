# Final branch review prompt (Phase 5)

For a fresh Opus reviewer agent (`subagent_type: "ecoskills:reviewer"`, or `"general-purpose"` if that type isn't available): `model: "opus"`, `effort: "medium"`. Fill in the `<...>` parts.

```markdown
# Request: Review the whole branch against its plan

All units of a plan have been implemented on branch <branch>, each reviewed individually. Per-unit reviews can't see problems that only appear across units. Review the branch as a whole.
Don't edit any files except your findings file. You may read the repository and run read-only commands, including the plan's verification commands, in <worktree path>. Don't run git commands that modify state.
<Command wrapper and scope notes, as in the per-unit review prompt.>
The plan document is committed on the branch only for execution and will be deleted before integration. Point out knowledge that exists only in the plan but should stay in the repository: conventions, procedures, and policies that the code relies on.

## Inputs
- Plan: <worktree path>/<plan path> (goals, success criteria, decisions, tasks)
- Diff: `git -C <worktree path> diff <base-sha>...HEAD`. Commits: `git -C <worktree path> log --oneline <base-sha>..HEAD`
- Execution log, with deviations and rejected review findings: <scratchpad>/execution-log.md
- Decisions made with the user during execution: <list, or "recorded in the plan">

## What to look for
1. Goals and success criteria: is each one met? Run the plan's overall verification.
2. Plan conformance: decisions and constraints honored across the whole diff, including each long-term guideline's conformance checks, and no unplanned changes.
3. Cross-unit consistency: naming, structure, duplicated or conflicting changes, and an earlier unit's work undone by a later one.
4. Leftovers: debug code, temporary files, stale references to removed or renamed things, TODOs.
5. Correctness and quality: bugs, fragile constructs, and comments or docs that no longer match the code.
6. Deviations recorded in the log: are they justified, and are they reflected wherever they matter?

## Output format
Write your findings to <scratchpad>/final-review.md and return them as your final message.
List findings by severity (high / medium / low). For each: where, the problem, evidence (command and output, or quoted lines), and a suggested fix.
End with a verdict: "ready to integrate" or "not ready", plus the blocking findings.
```
