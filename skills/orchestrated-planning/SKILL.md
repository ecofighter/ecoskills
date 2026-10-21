---
name: orchestrated-planning
description: Orchestrates planning for a substantial task — analyze it from multiple angles, gather and organize context, delegate design and research to model-matched subagents (Opus for short-to-mid-term work, Fable for long-term guidelines), put each result through adversarial review by a fresh Sonnet agent and revision, then consolidate everything into a structured document under docs/plans/ from which an implementation agent can build a rigorous implementation plan. Use this whenever the user asks to analyze a task thoroughly before implementing, decide architecture or development guidelines, split research and design across subagents, or produce a plan document for a sizable feature, migration, or refactor — even if they don't name this skill (e.g. 「多角的に分析して計画を立てて」「設計方針を決めたい」「調査と設計をサブエージェントに分けて」「実装前にちゃんと計画書を作りたい」). Do not use for single-file fixes or work whose plan is already settled and ready to implement.
---

# Orchestrated Planning

Process a substantial task through analysis → context gathering → delegation → adversarial review → revision → consolidation, and produce **a document from which an implementation agent can build a rigorous implementation plan without further investigation**.

The deliverable of this skill is the plan document. Do not implement anything.

## Overview

```
Phase 1  Multi-angle analysis          (this session)
Phase 2  Context gathering             (this session or Sonnet agents) — no value judgments
Phase 3  Decomposition and assignment  (this session) → user approval
Phase 4  Delegation                    (Opus/medium or Fable/medium) → suspend after response
Phase 5  Adversarial review            (fresh Sonnet agent)
Phase 6  Resume and revise             (resume the Phase 4 agent via SendMessage)
Phase 7  Consolidation                 (this session) → docs/plans/
```

The core of this skill is keeping each phase's role separate. In particular, if value judgments leak into Phase 2, downstream agents receive the orchestrator's biases as premises and can no longer make independent design decisions.

Save intermediate artifacts (analysis memo, context dossier, each briefing, each agent's output, review results, agent IDs) as files in one work directory: the scratchpad directory (or `docs/plans/.work/<slug>/` if there is none). This lets work continue after the conversation context gets summarized, and lets agents exchange artifacts by path instead of having the orchestrator re-type them. Suggested names: `01-dossier.md`, `S1-briefing.md`, `S1-response-r1.md`, `S1-review-r1.md`, `S1-response-r2.md`, `agents.txt`.

Write the final plan document in the language the user is working in.

---

## Phase 1: Multi-angle analysis

Before doing anything else, analyze the task from the following angles and record the result as an analysis memo. Not every angle applies; pick the ones relevant to the task.

- **Goal**: What counts as success? What is the real purpose behind the request?
- **Scope**: What is included / explicitly excluded?
- **Time horizon**: Do the consequences of the decisions close within days to months, or last longer? (This drives the assignment in Phase 3.)
- **Reversibility**: How costly is it to change later? (Data models, public APIs, dependencies, and conventions lean irreversible.)
- **Technical constraints**: Existing code, platform, performance, security.
- **Stakeholders and users**: Who uses it, who maintains it?
- **Dependencies**: Which decisions are prerequisites for others?
- **Unknowns**: What can't be known without investigation? (These become the Phase 2 gathering targets.)

**Confirm intent with the user when the goal is ambiguous.** A request like "tidy up the Linux config" can mean several different projects, and the decomposition in Phase 3 depends entirely on which one. Do a quick first pass of Phase 2 (enough to know what the options are), then ask the user the few questions whose answers change the decomposition: which goals matter, upcoming plans that set the time horizon (e.g. "will more machines be added?"), and whether parts of the system are still in use. Offer concrete options derived from the facts you found rather than open-ended questions. Record the answers in the dossier as premises.

If the analysis shows the task is small and needs only a single short-term decision, tell the user the full pipeline is overkill and ask whether to proceed in a lighter form.

## Phase 2: Context gathering and organization

Gather the unknowns from Phase 1 and the information downstream agents will need to make their decisions.

**Who gathers**
- A few file reads or information already in the conversation is enough → gather it directly in this session.
- Broad code exploration, research across multiple areas, or web research is needed → delegate to Sonnet agents (`Agent` with `model: "sonnet"`). Split by independent research area and launch them in parallel in a single message.

When delegating to Sonnet, the prompt should include:
- What to investigate and which decision the information will feed into (stating that the decision itself is not being asked for).
- **Stick to collecting and organizing facts; do not write evaluations, recommendations, or rankings.**
- Attach a source to each fact (`file:line`, URL, command output).
- Distinguish "confirmed facts", "inferences", and "could not confirm".

**How to organize**

Consolidate the gathered information into a single "context dossier". Still no value judgments at this stage: don't write good/bad or recommended/not recommended; classify and list facts, constraints, existing decisions, and unconfirmed items.

Why avoid value judgments: if the dossier says "A looks better", the Phase 4 agent treats that as a premise and stops examining the question independently. Leave judgment to the agents responsible for it.

## Phase 3: Decomposition and assignment

Using the context dossier, carve out subtasks to delegate. Size each subtask so that it "answers one question".

**Assignment criteria**

| Assignee | Scope | Examples |
|---|---|---|
| **Fable / effort medium** | Design with long-term impact, tasks that set development guidelines, and research feeding into them | Architecture direction, data models, public interfaces, choice of core dependencies, development guidelines such as coding conventions or test strategy, migration strategy |
| **Opus / effort medium** | Short-to-mid-term design and research | Concrete design within established guidelines, module-level design, investigating a specific bug or behavior, comparing a bounded set of options |

Tie-breaker when unsure: "If this decision were reversed six months from now, how much other work would have to be redone?" Widespread rework → Fable; localized → Opus.

**Handling dependencies**
- If a Fable guideline is a prerequisite for an Opus design, run the Fable task to completion (through Phase 6) first, and include its revised conclusion in the Opus task's briefing.
- Launch mutually independent tasks in parallel.

**User approval**

Before delegating, present the following to the user and get approval. Fable is expensive, so assignment mistakes should be caught before launch.
- The subtask list (question, assigned model, reason for the assignment, dependencies, execution order).
- Key points of the context dossier, and anything that will be delegated while still unresolved.

Apply any corrections before proceeding.

## Phase 4: Delegation

Write a briefing for each subtask and launch an agent. Follow the structure in `references/briefing-template.md`.

**Briefing principle: neither too much nor too little**
- Pass enough context, extracted from the dossier, that the agent **can decide without investigating on its own**. Don't paste the whole dossier; select what's relevant to this task.
- Quote the relevant code and configuration, not just their paths.
- Leave out unrelated information. Noise degrades judgment.
- State already-settled matters (user instructions, conclusions of prior tasks) as "premises" and tell the agent not to relitigate them.

**How to launch**

```
Agent(
  subagent_type: "general-purpose",
  model: "opus" | "fable",
  effort: "medium",
  description: "<3-5 word description>",
  prompt: <briefing>
)
```

Write the briefing to the work directory first (`S<n>-briefing.md`); the prompt can then simply point the agent at that file. Launch independent tasks in parallel in a single message. **Record each agent's ID** in `agents.txt` (needed to resume it in Phase 6).

Two instructions belong in every briefing (they are in the template):
- **Write the final deliverable to a given file path** (`S<n>-response-r<round>.md`) as well as returning it. Otherwise the orchestrator has to re-type or extract it before the reviewer can read it.
- **Verify cheaply where possible and tag claims** as verified (by evaluation, test run, or reading the source) or inferred. Allow read-only checks and experiments in a scratch copy of the repository, never in the repository itself. Agents that can check their own claims produce far fewer unsupported assertions, and the tags tell the reviewer where to push.

Once an agent responds, give it no further instructions and leave it idle (= suspended).

**Coordinating parallel subtasks.** Sometimes two parallel subtasks touch the same cross-cutting decision (e.g. one designs features, the other decides which environments receive them). Decide up front which subtask owns that decision. Then, when resuming them in Phase 6, pass each one the other's relevant proposals as facts. Tell the non-owner to make its design work under any outcome of the owner's decision (for example, by isolating the affected part behind a single switch). This avoids a contradiction that would otherwise only surface in Phase 7.

## Phase 5: Adversarial review

For each subagent response, launch **one Sonnet agent with a fresh context** to review it (`model: "sonnet"`). Launch them in parallel when there are several.

Why a fresh context: a reviewer sharing context with this session or the original agent tends to accept the same premises uncritically. Give the reviewer only the briefing and the response, by file path, and never the orchestrator's impressions. Let the reviewer check claims against the repository read-only, and tell it to save its findings to `S<n>-review-r<round>.md`.

Use the template in `references/reviewer-prompt.md` for the reviewer prompt. Findings are expressed in three forms:
- **Open questions**: unsupported claims, logical leaps, ambiguous statements.
- **Implementation challenges**: points where an implementer would get stuck starting from this as-is, undefined parts, missing steps.
- **Soundness concerns**: validity of premises, contradictions, overlooked risks or alternatives, inconsistency with the briefing's constraints.

## Phase 6: Resume and revise

Use `SendMessage` addressed to the Phase 4 agent's ID to send the review results and resume it with its context intact. (If `SendMessage` is a deferred tool, load it first with `ToolSearch` using `select:SendMessage`.)

The resume instruction should include:
- The path of the review file, and any coordination facts from parallel subtasks (see Phase 4).
- Assess the validity of each finding and mark it **accepted / partially accepted / rejected** with reasons. The agent should feel free to reject findings where the reviewer is wrong.
- Revise conclusions and plans based on accepted findings.
- Add information **only as far as the implementer needs it to decide and act**; don't become overly explanatory. Fill in missing decisions, conditions, and concrete values rather than adding background explanation or restatements.
- Output in two parts: "the revised deliverable (in full)" and "a response table for the findings".

**When to do a second round**: Repeat Phases 5–6 once more only if a serious finding (infeasible to implement, clear contradiction, constraint violation) was rejected without good reason, or the revision introduced a new serious problem. Do not go beyond two rounds; carry anything left over into Phase 7 as unresolved.

**Fallback**: If `SendMessage` is unavailable or the agent has terminated and can't be resumed, launch a new agent with the same model and effort, pass it "the original briefing + the previous response + the review results", and give it the same instructions.

Start dependent downstream tasks from Phase 4 only after their prerequisite tasks have completed Phase 6.

## Phase 7: Consolidation into the plan document

Consolidate all subtasks' revised deliverables and save to `docs/plans/YYYY-MM-DD-<topic>.md`. Follow the structure in `references/plan-document-template.md`.

**During consolidation**
- Detect contradictions and overlaps between subtask conclusions. Resolve the ones the orchestrator can resolve on solid grounds and record the reasoning; ask the user about ones that come down to differing value judgments.
- Keep long-term guidelines (from Fable) separate from short-to-mid-term design (from Opus), so the implementer knows what to treat as fixed constraints.
- Link every implementation task to the IDs of the decisions it is based on. An implementation task with no linked decision signals a missing decision.
- Size implementation tasks so that each one fits in a single commit, and give each one exact verification commands with expected results. The plan is executed by the `orchestrated-execution` skill, which commits and reviews one task at a time.
- Inline what the implementer would otherwise have to look up: code that must be recreated, exact snippets, and commands. The implementer should not need this skill's work directory, because it will not exist later.

**Document standard: neither too much nor too little**

The reader is the "implementation agent that will next direct subagents to implement this". That agent does not have this conversation's context. The document must satisfy:
- From this document alone, the purpose, targets, completion criteria, and order of each implementation task are clear.
- Wherever a judgment call will arise during implementation, there is either a settled policy or an "undecided; under this condition, do this" default.
- Discussion history and details of rejected alternatives are included only as far as needed to keep the implementer from relitigating them.

**Final check**

Before saving, reread the document from the implementation agent's point of view and confirm:
- [ ] Every implementation task has verifiable completion criteria.
- [ ] Dependencies and execution order can be read unambiguously.
- [ ] Every unresolved item says how it will be handled (who, when, default if still unresolved).
- [ ] Nothing that exists only in the conversation (e.g. verbal user instructions) has been left out.
- [ ] No explanation or history unnecessary for implementation remains.

## Completion report

Tell the user, concisely:
- The path to the plan document.
- A summary of the key decisions (especially long-term guidelines).
- What changed significantly as a result of the review.
- Unresolved items, and which ones need the user's decision.
- Next step: offer to execute the plan with the `orchestrated-execution` skill.
