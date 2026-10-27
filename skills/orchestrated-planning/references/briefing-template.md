# Briefing template (Phase 4)

Structure of the prompt passed to an Opus / Fable subagent (`subagent_type: "ecoskills:planner"`, or `"general-purpose"` if that type isn't available). Sections may be omitted as appropriate, but "Question", "Premises", and "Output format" are always required.

```markdown
# Request: <the subtask's question in one sentence>

## Your role
<e.g. Decide as the designer responsible for this project's long-term development guidelines.>
Your output will go through adversarial review and then be consolidated into a plan document for an implementation agent.
Do not edit the repository. For checks or experiments, use a scratch copy at <scratch path>.
Write your final deliverable to <work dir>/S<n>-response-r1.md and also return it as your final message.

## Overall goal
<The goal of the whole task and the part this subtask plays in it. 2-4 sentences.>

## Question
<The question(s) to answer. Number them if there are several. State the expected granularity of the decision.>

## Premises (settled — do not relitigate)
- <User instructions, conclusions of prior tasks, immovable constraints>

## Context
This section is organized so you can decide from it alone. In principle no additional investigation is needed.
If something is missing and you do a minimal check yourself, record what you checked under "Additional checks" in your output.

### Current state
<Relevant code, configuration, structure. Quote what's needed and cite sources (file:line).>

### Constraints
<Technical and operational constraints>

### Facts relevant to the decision
<Facts from the investigation, with sources. No evaluations.>

### Unconfirmed items
<Things that could not be confirmed. Tell the agent to treat any premise placed on them as an assumption.>

## Out of scope
<What this subtask does not cover. Note if another task covers it.>

## Output format
Output in the following structure. Don't be overly explanatory; prioritize information the implementer can use to decide and act.
Where a claim can be checked cheaply (evaluation, a test run, reading the source), check it. Tag each factual claim as [V] (verified, and say how) or [I] (inferred).
Completion criteria should be exact commands with exact expected results.

1. **Conclusion**: the answer to the question (bullet points)
2. **Rationale**: reasons supporting the conclusion, and which facts in the context they rest on
3. **Alternatives considered and rejected**: each option with 1-2 lines on why it was rejected
4. **Assumptions made**: assumptions about unconfirmed items, and the impact if they turn out false
5. **Implications for implementation**: constraints the implementer must follow, concrete units of work, candidate completion criteria
6. **Unresolved items**: points you couldn't decide, and the default if work proceeds with them unresolved
7. **Additional checks**: (if any) what you investigated yourself
```

## Notes on writing

- Select **only the parts of the context dossier relevant to this task**. Don't paste the whole dossier.
- If code or configuration matters to the decision, quote the relevant section, not just the path.
- Don't mix the orchestrator's opinions (e.g. "X seems better") into the context.
- Give Fable richer context about long-term scope of impact (future extensions, maintainers, other systems); give Opus richer context about the concrete current state (code, existing guidelines).
