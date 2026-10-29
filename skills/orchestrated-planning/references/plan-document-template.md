# Plan document template (Phase 7)

Save to: `docs/plans/YYYY-MM-DD-<topic>.md`

The reader is an implementation agent that will direct subagents to implement this, without access to this conversation's context or the work directory. The goal is that this document alone is enough to implement the plan rigorously: the implementer commits one task at a time, has each commit reviewed against this document, and deletes the document before integrating. Sections that don't apply may be removed, but "Decisions", "Implementation tasks", "Execution order", and "Verification plan" are required.

Write the document in the language the user is working in; translate the headings below accordingly.

```markdown
# <Title>

- Created: YYYY-MM-DD
- Base: <branch> @ <short SHA> (the state that "Background context" and all file:line references describe)
- Status: Plan finalized / Some items unresolved (see 10)

## 1. Goal and success criteria
- Goal: <what this achieves>
- Success criteria: <in a verifiable form; point to the checks in 9>

## 2. Scope
- In scope: <what is included>
- Out of scope: <what is excluded; note if it will be handled later>

## 3. Background context
Facts about the current state the implementer needs to know, with sources.
- <fact> (`path/to/file:line`)

## 4. Prerequisites
What implementation and verification need from the environment. The implementer checks these before the first change.

| Needed | For | How to check | If missing |
|---|---|---|---|
| <tool, credential, service, hardware, access> | <T-x / verification> | `<command>` → <expected> | <stop and ask / skip T-x / alternative> |

## 5. Decisions

### 5.1 Long-term guidelines (changing these requires re-examination)
If implementation runs into a conflict with any of these, stop and report.

#### D-1: <decision title>
- Decision: <what was decided>
- Rationale: <why>
- Rejected alternatives: <option — reason in one line>
- Scope of impact: <where it applies>
- Revisit when: <conditions that should trigger re-examination>
- Conformance checks: <each MUST / MUST NOT rule, and how to tell from the code whether it is followed>
- Kept in the repository: <where it is recorded once this plan is deleted, and which task writes it (e.g. "CONTRIBUTING.md, by T-5")> / not needed

### 5.2 Short-to-mid-term design
May be adjusted during implementation for good reason, but record any adjustment.

#### D-10: <decision title>
- Decision / Rationale / Rejected alternatives (as above, only as much as needed)

## 6. Implementation constraints and conventions
- <rules the implementer must follow, each tagged with the decision it derives from (D-x)>

## 7. Implementation tasks
Each task is one commit and can be verified on its own.

| ID | Task | Based on | Depends on | Parallelizable |
|---|---|---|---|---|
| T-1 | <summary> | D-1, D-10 | - | yes |
| T-2 | <summary> | D-11 | T-1 | no |

### T-1: <task name>
- Purpose: <why>
- Targets: <files / modules>
- Work: <what to do. Quote exact snippets, mappings, and commands where the plan fixes them. Refer to decisions wherever a judgment call is involved.>
- Left to the implementer: none / <each choice the plan deliberately leaves open, with the criterion for choosing>
- Condition: always / <when this task applies (e.g. "only if Q-1 is answered yes"), and what to do otherwise>
- Completion criteria:
  - `<exact command>` → <expected result (exit code, output, or observable state)>
  - (only if no command can check it) Manual: <what to check, by whom, on what environment>
- Pitfalls: <traps raised in review, etc.>

## 8. Execution order
<Dependency graph or phases. Make parallelizable groups explicit.>

## 9. Verification plan
- Baseline (before the first change): `<command>` → <result expected at Base, including known pre-existing failures> / none
- Overall verification (after all tasks): `<exact command>` → <expected result>
- Manual checks by the user: <what, on what environment, and which success criterion it covers> / none

## 10. Risks and unresolved items

| ID | Item | Impact | How / when it gets resolved | Default if unresolved |
|---|---|---|---|---|
| Q-1 | <item> | <affects T-x> | <who decides, and when> | <what to do if it isn't decided> |

## Appendix: Main changes from review
Only what's needed to prevent relitigation.
- <gist of the finding> → <accepted/rejected, and the outcome>
```

## Notes on writing

- Using separate ID ranges for long-term decisions (D-1…) and short-to-mid-term ones (D-10… etc.) helps the implementer tell their binding force apart.
- Every implementation task must reference its decisions. A task with no reference either lacks a decision or is unnecessary.
- Avoid phrases that hand the judgment to the implementer, like "as appropriate" or "if needed". If a judgment is required, state the criterion. "Left to the implementer" is the one place for choices that are deliberately open; it also tells the implementer how much judgment, and so how capable an executor, each task needs.
- Completion criteria must be runnable as written. "Tests pass" is not a criterion; `nix flake check` → exit 0, or `pytest tests/test_x.py` → 12 passed, is.
- The plan is deleted after implementation. Snippets meant for the code must not cite plan IDs (D-x, T-x) or the plan document. Anything the code will keep relying on (conventions, maintenance procedures, policies) needs a task that writes it into the repository.
- Include history only as far as needed to keep the implementer from repeating the same discussion.
