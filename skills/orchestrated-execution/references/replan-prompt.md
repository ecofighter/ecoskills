# Re-plan prompt template (Phase 4.4)

Use this when execution has stopped on a problem that needs the plan revised. Show the filled-in prompt to the user and run it only on their go-ahead. It is written so that the `orchestrated-planning` skill can run from it without this conversation's context. Run it in the main session (by invoking the skill with this prompt as its argument), because planning asks the user to approve its task assignment. If the user prefers a subagent, the subagent must be able to spawn agents, and its approval questions must be relayed to the user.

```markdown
Use the orchestrated-planning skill to revise an existing plan whose execution hit a blocking problem.

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
A revised plan document at <path> (same structure as the original), with the changed decisions and tasks marked, and a short summary of what changed and why. Execution will resume from Phase 1 of the orchestrated-execution skill on that plan.
```
