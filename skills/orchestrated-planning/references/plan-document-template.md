# Plan document template (Phase 7)

Save to: `docs/plans/YYYY-MM-DD-<topic>.md`

The reader is an implementation agent that will direct subagents to implement this, without access to this conversation's context. The goal is that this document alone is enough to build a rigorous implementation plan. Sections that don't apply may be removed, but "Decisions", "Implementation tasks", and "Execution order" are required.

Write the document in the language the user is working in; translate the headings below accordingly.

```markdown
# <Title>

- Created: YYYY-MM-DD
- Status: Plan finalized / Some items unresolved

## 1. Goal and success criteria
- Goal: <what this achieves>
- Success criteria: <in a verifiable form>

## 2. Scope
- In scope: <what is included>
- Out of scope: <what is excluded; note if it will be handled later>

## 3. Background context
Facts about the current state the implementer needs to know, with sources.
- <fact> (`path/to/file:line`)

## 4. Decisions

### 4.1 Long-term guidelines (changing these requires re-examination)
If implementation runs into a conflict with any of these, stop and report.

#### D-1: <decision title>
- Decision: <what was decided>
- Rationale: <why>
- Rejected alternatives: <option — reason in one line>
- Scope of impact: <where it applies>
- Revisit when: <conditions that should trigger re-examination>

### 4.2 Short-to-mid-term design
May be adjusted during implementation for good reason, but record any adjustment.

#### D-10: <decision title>
- Decision / Rationale / Rejected alternatives (as above, only as much as needed)

## 5. Implementation constraints and conventions
- <rules the implementer must follow, each tagged with the decision it derives from (D-x)>

## 6. Implementation tasks

| ID | Task | Based on | Depends on | Parallelizable |
|---|---|---|---|---|
| T-1 | <summary> | D-1, D-10 | - | yes |
| T-2 | <summary> | D-11 | T-1 | no |

### T-1: <task name>
- Purpose: <why>
- Targets: <files / modules>
- Work: <what to do; refer to decisions wherever a judgment call is involved>
- Completion criteria: <testable; which tests passing means done>
- Pitfalls: <traps raised in review, etc.>

## 7. Execution order
<Dependency graph or phases. Make parallelizable groups explicit.>

## 8. Verification plan
- <How the whole is verified: tests, manual checks, measurements>

## 9. Risks and unresolved items

| ID | Item | Impact | How / when it gets resolved | Default if unresolved |
|---|---|---|---|---|
| U-1 | <item> | <affects T-x> | <who decides, and when> | <what to do if it isn't decided> |

## Appendix: Main changes from review
Only what's needed to prevent relitigation.
- <gist of the finding> → <accepted/rejected, and the outcome>
```

## Notes on writing

- Using separate ID ranges for long-term decisions (D-1…) and short-to-mid-term ones (D-10… etc.) helps the implementer tell their binding force apart.
- Every implementation task must reference its decisions. A task with no reference either lacks a decision or is unnecessary.
- Avoid phrases that hand the judgment to the implementer, like "as appropriate" or "if needed". If a judgment is required, state the criterion.
- Include history only as far as needed to keep the implementer from repeating the same discussion.
