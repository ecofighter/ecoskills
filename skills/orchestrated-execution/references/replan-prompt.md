# Re-plan request template (Phase 4.4)

Use this when execution has stopped on a problem that needs the plan revised. Show the filled-in request to the user and act on it only on their go-ahead. It must work without this conversation's context, whether the user hands it to a planning skill or workflow, to a separate session, or revises the plan here. If the user names a planning skill or workflow, add a first line asking for it.

```markdown
Revise an existing plan document whose execution hit a blocking problem.

## Plan being revised
- Plan document: <absolute path in the worktree> (committed on branch <branch>)
- Repository / worktree: <absolute worktree path>. Work only from this tree; it contains the completed units.

## Execution status
| Unit | Plan task | Status | Commit |
|---|---|---|---|
| U-1 | T-1 | done | <sha> |
| ... | ... | ... | ... |
| U-k | T-k | stopped here | <sha or "uncommitted"> |

## The problem
- What happened: <concrete description>
- Evidence: <commands and output, file:line>
- Why it blocks the plan: <which decision, premise, or criterion is wrong>
- Remaining units affected: <list>, and units not affected: <list>

## Decisions already made with the user
<Phase 1 answers and any later decisions, verbatim>

## Constraints on the revision
- Keep completed units unless the problem requires changing them. If it does, state which ones and how.
- Revise only the affected decisions and tasks; leave the rest of the plan intact.
- <any user-stated constraint>

## Expected output
A revised plan document at <path>, with the same structure as the original, the changed decisions and tasks marked, and a short summary of what changed and why. Each remaining task still needs exact completion commands with expected results. Update the plan's recorded base commit (if it has one) and its file:line references to the current branch HEAD, which includes the completed units. Execution resumes from the revised plan's first unfinished task.
```
