# Adversarial review prompt (Phase 5)

Pass this to a Sonnet agent with a fresh context (`model: "sonnet"`). Fill in the `<...>` placeholders. Do not pass the orchestrator's impressions or other subtasks' results.

```markdown
# Request: Adversarial review of a design / research result

You are a reviewer in the position of the person who will be asked to implement based on the deliverable below.
This deliverable will become the foundation of an implementation plan. Problems not caught here turn into rework during implementation.
There is no need to point out strengths, and no need to rewrite anything. Focus on identifying problems.

Do not edit any files except your findings file. You may read the repository at <repo path> and run read-only checks against it to verify claims made in the deliverable.
Claims tagged [I] in the deliverable are inferred. Push hardest on them, and on untagged claims.
Write your findings to <work dir>/S<n>-review-r<round>.md and also return them as your final message.

## Original request (briefing)
Read in full: <work dir>/S<n>-briefing.md (and any settled-premise file it names)

## Deliverable
Read in full: <work dir>/S<n>-response-r<round>.md

## What to look for
Express findings in the following three forms.

### A. Open questions
- Claims with no stated basis, or where the basis doesn't connect to the conclusion
- Ambiguous statements open to multiple interpretations
- Places that rely on facts not present in the briefing's context

### B. Implementation challenges
- Points where starting implementation as-is would leave you stuck on a decision; undefined parts
- Missing steps, ordering, or completion criteria
- Likely problems connecting to existing code or operations

### C. Soundness concerns
- Validity of premises and assumptions
- Internal contradictions; inconsistency with the briefing's premises or constraints
- Overlooked risks, failure modes, or alternatives that should have been considered
- Decisions that could become problems long-term (maintainability, extensibility, migration cost)

## Output format
Report each finding in the following format, ordered by severity, highest first.

- **[A|B|C]-<number>** Severity: high / medium / low
  - Target: <which part of the deliverable (quote it)>
  - Finding: <the problem, phrased as a question or a concrete concern>
  - Reason: <why it is a problem>

Severity guide:
- High: infeasible to implement, clear contradiction, constraint violation, overlooked serious risk
- Medium: forces the implementer to make a judgment call; possible rework
- Low: worth clarifying, but work can proceed without it

Be specific. Avoid vague findings like "add more detail"; state what is missing and what problem it causes.
For any category where you find no problems, write "No findings". Do not pad the list.
```
