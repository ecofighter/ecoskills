# Guideline conformance check prompt (Phase 6.5)

Pass this to a fresh reviewer agent (`subagent_type: "ecoskills:reviewer"`, or `"general-purpose"` if that type isn't available; `model: "sonnet"`). Fill in the `<...>` placeholders. Do not pass the orchestrator's impressions or the earlier review findings.

```markdown
# Request: Check a set of designs against the long-term guidelines

Several subtasks of one plan have been designed and reviewed one by one. They will be consolidated into a single plan document next.
Each was reviewed on its own, so problems that appear only when they are combined, or that crept in during revision, have not been checked. Find those.
There is no need to point out strengths, and no need to rewrite anything.

Do not edit any files except your findings file. You may read the repository at <repo path> and run read-only checks against it.
Write your findings to <work dir>/X-conformance-r<round>.md and also return them as your final message.

## Guidelines
Read in full: <work dir>/S<m>-response-r<k>.md <, ...>
The conformance criteria (G-<n>) in these are what you check against. Each states its rationale ("Why"), its scope ("Applies to"), and how to check it.

## Deliverables to check
Read in full:
- <work dir>/S<n>-response-r<k>.md — <the subtask's question in one line>
- ...

## What to look for
- Each deliverable against every criterion whose scope covers it.
- The deliverables combined: a guideline broken only by their combination (e.g. two designs that each conform but together create the dependency or duplication a criterion forbids).
- Disagreements between deliverables (names, interfaces, values, ownership, ordering), even where no criterion applies.

## Output format
Classify each finding as exactly one kind:
- **Violation**: the deliverable clearly breaks a criterion as written.
- **Interpretation**: whether it conforms depends on how a criterion is read, or the criterion's scope does not settle the case. Say which readings are possible.
- **Challenge**: a deliverable gives concrete evidence (a verified fact, not a preference) that a criterion cannot or should not hold.
- **Conflict**: two deliverables disagree on something no criterion covers.

Report each finding as follows, ordered by severity, highest first:

- **[V|I|Ch|Cf]-<number>** Severity: high / medium / low
  - Criterion: <G-n, or "none" for a conflict>
  - Target: <deliverable path and the quoted part(s)>
  - Finding: <the concrete problem>
  - Evidence: <why it breaks or strains the criterion; for a challenge, the fact and its source>

Severity guide:
- High: the plan would build something that breaks a long-term guideline, or two parts cannot both be implemented
- Medium: the implementer would have to make the call; likely rework
- Low: worth aligning, but nothing breaks

Prefer Violation when the criterion's wording decides the case; use Interpretation only when it genuinely does not.
Write "No findings" if there are none. Do not pad the list.
```
