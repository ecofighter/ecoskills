# Per-unit adversarial review prompt (Phase 4.2)

For a fresh agent matched to the unit's tier: `model: "sonnet"` (standard) or `model: "opus"` (hard), with `effort: "medium"`. Fill in the `<...>` parts. Don't include your own assessment of the unit.

```markdown
# Request: Adversarial review of one implemented unit

A plan document is being implemented one commit at a time. Review the commit below. The work continues from this commit, so problems you miss turn into rework later.
There is no need to point out strengths. Don't edit any files except your findings file. You may read the repository and run read-only commands (including the plan's verification commands) in <worktree path>. Don't run git commands that modify state. Keep Bash commands simple and run them from that directory.
<If commands are restricted in the worktree: "Use the helper <wrapper path> <usage> instead of <blocked command>.">
<Scope notes, e.g.: "Commit trailers (Co-Authored-By etc.) follow the session's attribution instruction and are out of scope.">
If you can't write the findings file, return the findings in your final message only.

## Inputs
- Plan: <worktree path>/<plan path>. The unit implements section <section>. The next unit is section <section>.
- Commit: <sha>. Inspect it with `git -C <worktree path> show <sha>`.
- Verification output reported by the executor: <path to report file, or inline>
- Advisor consultations for this unit: <advice file paths, or "none">. Choices that follow this advice were deliberate; flag them only if they contradict the plan.
- Choices the plan explicitly leaves to the implementer are sanctioned. Flag them only if the chosen option breaks a criterion the plan gives for that choice.

## What to look for
### A. Contradictions with the plan
- Deviations from the plan's decisions, constraints, or this unit's specified contents
- Completion criteria not actually met, or met only by weakening the check. Re-run them if in doubt.
- Changes outside this unit's scope

### B. Problems for the next unit
- Anything in this commit that blocks or complicates the next unit
- Premises of the next unit (files, names, interfaces, values) that this commit changed or did not set up as the plan expects
- State left behind (temporary files, partial edits) that the next unit would trip over

## Output format
Write your findings to <scratchpad>/<U-n>-review.md and return them as your final message.
Order them by severity, highest first:

- **[A|B]-<n>** Severity: high / medium / low
  - Where: <file:line or plan section>
  - Finding: <the concrete problem>
  - Evidence: <command and output, or quoted lines>

Severity:
- high: the plan is violated, or the next unit cannot proceed
- medium: likely rework, or a judgment call the plan didn't sanction
- low: worth fixing, but nothing depends on it

Write "No findings" for a category with none. Don't pad the list.
If you find that the plan itself cannot be followed as written, say so explicitly as a separate note.
```
