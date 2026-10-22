---
name: orchestrated-execution
description: Executes a structured plan document (typically one under docs/plans/ that lists decisions, tasks, and completion criteria) task by task in a dedicated branch and git worktree — clarifying open points with the user first, splitting the plan into commit-sized units, delegating tasks to agents matched to each task's difficulty (Sonnet for precisely specified work, Opus for tasks that need judgment or subtle correctness) when context isolation helps, consulting an Opus or Fable advisor agent when a task hits a hard problem, putting every task through adversarial review by a fresh agent of the same tier, stopping to consult the user (and proposing a re-plan) when a serious contradiction surfaces, getting a final Opus review of the whole branch, and finally removing the plan document and merging or opening a PR. Use this whenever the user asks to implement, execute, or carry out a plan document, or says things like 「計画を実行して」「この計画書どおりに実装して」「docs/plans の計画を進めて」, even if they don't name this skill. Do not use for ad-hoc changes with no plan document.
---

# Orchestrated Execution

Carry out a plan document faithfully, one commit-sized task at a time, with an independent review after every task and a review of the whole at the end. The plan is the specification. The job is to implement it, not to redesign it. When the plan turns out to be wrong in a way that matters, stop and bring it back to the user instead of improvising.

## Overview

```
Phase 1  Analyze the plan and confirm open points   (this session) → user answers
Phase 2  Split into ordered, commit-sized units     (this session)
Phase 3  Branch + worktree                          (this session)
Phase 4  Execute each unit                          (this session, or a Sonnet/Opus agent by difficulty)
           └ hard problem mid-unit → consult an advisor (Opus, or Fable with the user's OK) → resume
           └ per unit: verify → commit → adversarial review (fresh agent of the unit's tier, effort medium) → fix
           └ serious problem → stop, consult the user, propose a re-plan
Phase 5  Final review of the whole branch           (fresh Opus, effort medium) → fix
Phase 6  Remove the plan, merge or open a PR, clean up the worktree (each step confirmed with the user)
```

Keep an **execution log** in the scratchpad directory: one row per unit with its status, difficulty tier, commit SHA, executor (self, or model and agent ID), review file, and outcome. Save every delegation brief and review result next to it as files. This lets work resume after the conversation context is summarized, and lets agents read artifacts by path.

**Agent types.** The ecoskills plugin ships one agent per delegated role:

| Role | Agent type | Tools |
|---|---|---|
| Executor (4.1) | `ecoskills:implementer` | Read, Grep, Glob, Bash, Edit, Write, WebFetch, WebSearch |
| Advisor (4.1) | `ecoskills:advisor` | Read, Grep, Glob, Bash, Write, Agent, WebFetch, WebSearch |
| Reviewer (4.2, Phase 5) | `ecoskills:reviewer` | Read, Grep, Glob, Bash, Write |

Each carries only the tools its role needs, so it starts with a far smaller context than `general-purpose` and does the same work for roughly half the tokens. Pass `model` (and `effort` where the phase gives one) on every launch; the agent's own default is only a fallback.

Use `general-purpose` instead, with the same prompt, model, and effort, in two cases:
- **The unit needs a tool the agent lacks.** Examples: a browser to check a UI, an MCP server the plan's verification uses, notebook editing. Decide this when you write the brief, from the unit's contents and completion criteria, and note it in the log. If a delegated agent reports that it needs a missing tool, re-delegate the unit to `general-purpose`, giving it the original brief and the report.
- **The agent types aren't available** because the skill is installed without the plugin. The prompts built from `references/` are self-contained, so they work unchanged.

Note in the log which agent type each delegation used.

---

## Phase 1: Analyze the plan and confirm open points

Read the whole plan document before touching anything. Then check it from the implementer's position:

- **Unresolved items and defaults**: list every item the plan marks as unresolved, owner-to-decide, vetoable, or conditional, together with its stated default.
- **Ambiguities**: places where two reasonable implementers would do different things, wording like "as appropriate", and missing completion criteria.
- **Drift**: spot-check the plan's "background context" against the repository as it is now (key files, line references, the verification baseline). If the plan records the commit it was written against, `git diff <that-commit>..HEAD -- <files the plan cites>` shows what moved. If it also names a base branch that differs from the one checked out now, ask which one to build on. Plans go stale. A changed premise is cheaper to catch now than in task 5.
- **Preconditions**: tools, credentials, or environments the plan lists or its verification commands need, and whether they are available here. Run the checks the plan gives for them, and follow its stated fallback when something is missing.
- **Internal consistency**: tasks that contradict decisions, and dependencies that don't match the stated order.

Ask the user about the points whose answers change what gets built. Use concrete options, and put the plan's default first. Don't ask about things the plan has already decided; the plan is the agreed specification.

Record the answers **in the plan document itself**: update the unresolved-items table, and strike or mark vetoed items. Delegated agents and reviewers read the plan, not this conversation, so the plan must reflect the decisions.

## Phase 2: Split into ordered units

Turn the plan's tasks into an ordered list of **units**. Each unit is one feature-level change that belongs in a single commit and can be verified on its own.

- Usually the plan's tasks map one-to-one. Split a task that mixes independent changes, and merge tasks that cannot be verified separately.
- Respect the plan's execution order and dependencies. Drop units for vetoed items, and conditional units whose condition is unmet.
- For each unit, note the plan section it implements, its completion criteria, its difficulty tier (standard or hard, see 4.1), and whether you will execute it yourself or delegate it.

Show the user the unit list briefly, including the tier of each unit, and proceed, unless the split departs from the plan in a way the user should weigh in on. Mention the hard units explicitly: Opus costs more, so the user may want to move a unit to the other tier.

## Phase 3: Branch and worktree

1. Record the base branch (the branch checked out now) and its HEAD SHA. Phase 5 diffs against it.
2. Create a branch (e.g. `plan/<topic>`) and a new worktree for it, then move into it:
   - If the `EnterWorktree` tool is available (load it with `ToolSearch` if it is deferred), use it.
   - Otherwise, run `git worktree add <path> -b <branch> <base>` and work in that path, using absolute paths from then on.
3. Make the plan document available on the branch. If it is untracked or modified in the original tree, it won't appear in the new worktree. Copy it in and commit it as the branch's first commit, including the Phase 1 decisions. Every agent then reads the same version, and Phase 6 removes it with a commit.
4. Run any baseline step the plan prescribes (e.g. recording verification output) before the first change.
5. Check that the plan's verification commands actually run inside the worktree session. A worktree-isolated session may refuse commands it can't prove stay inside the worktree: compound commands, `cd`, or commands containing words like `eval` (which blocks `nix eval`). If a command is refused, run it as separate plain commands, or put it in a small wrapper script in the scratchpad that takes the varying parts as arguments. Tell every delegated agent and reviewer about the wrapper, since they hit the same restriction.

## Phase 4: Execute each unit

Work through the units in order. For each one:

### 4.1 Execute

**Doing it yourself vs. delegating.** Delegate (`Agent` with `subagent_type: "ecoskills:implementer"` and the model from the table below; see "Agent types" for when to use `general-purpose`) when context isolation pays off:
- the unit is self-contained and the plan specifies it well enough to work from the plan alone;
- or it requires reading many files, or produces long command output that would crowd this session's context.

Do it yourself when the unit is small, needs judgment that spans several units, or touches something that just changed because of a review.

**Choosing the model by difficulty.** The tier decides the executor's model and, in 4.2, the reviewer's model.

| Tier | Model | Units it fits |
|---|---|---|
| **standard** | Sonnet (`model: "sonnet"`) | The plan fixes the how, not just the what: exact snippets, file locations, commands. The change is local, mostly mechanical, and the completion criteria check it directly. Examples: adding a config entry, wiring a function the plan already designed, renaming across files per a given mapping, writing tests the plan enumerates. |
| **hard** | Opus (`model: "opus"`) | The executor has to make real decisions or find things out. The plan states the goal but leaves the implementation open, and the choices affect later units. Correctness is subtle: concurrency, security boundaries, data migration, state machines, error recovery, performance-sensitive code. The cause of a failure must be diagnosed in unfamiliar or tightly coupled code. A cross-cutting change must stay consistent across many modules. |

Tie-breaker when unsure: "Could a careful implementer finish this by following the plan literally, without decisions the plan doesn't make?" Yes → standard; no → hard. If the plan states per task which choices it leaves to the implementer, start from that: none usually means standard. Most units of a good plan are standard. If most units come out hard, the plan is probably underspecified; raise that in Phase 1 rather than paying for Opus throughout.

When delegating, write the brief from `references/delegation-brief.md` to a file and point the agent at it. Record the model and agent ID in the log; you may need to resume it in 4.3.

**Consulting an advisor.** An executor can hit a hard problem in the middle of a unit: a failure whose cause it can't find after a couple of real attempts, an implementation choice the plan leaves open with consequences beyond this unit, or two readings of the plan that lead to different code. It shouldn't guess or keep trying the same thing. Ask an advisor, a fresh agent that diagnoses and recommends but doesn't edit anything.

- **Who asks.** You launch every advisor, so that consultations stay visible in the log and the cost stays under your control. A delegated executor that hits such a problem stops, leaves its work uncommitted, and returns a consultation request (the brief tells it how). You run the consultation and resume the executor with `SendMessage`, passing the advice file path. When you are the executor, consult the same way.
- **Which model.** Launch the advisor as `subagent_type: "ecoskills:advisor"` with one of these models:

  | Advisor | Use it when |
  |---|---|
  | **Opus** (`model: "opus"`, `effort: "medium"`) | The problem stays inside this unit or the next one or two: a bug diagnosis, choosing an implementation approach, a confusing tool or library behavior. This is the default. |
  | **Fable** (`model: "fable"`, `effort: "medium"`) | The answer affects the plan's long-term guidelines, the overall architecture, or many remaining units. Also use it when an Opus advisor's advice didn't resolve the problem. Fable is expensive, so tell the user why you want it and get a go-ahead first. |

- **Keeping the advisor lean.** Write the request from `references/advisor-prompt.md` to a file. Pass paths and short excerpts rather than pasting logs and diffs. The prompt tells the advisor to save its own context: send broad searches, long logs, and reading many files to Sonnet or Haiku agents; use a `fork` (`subagent_type: "fork"`) when it wants to explore a hypothesis in depth without filling its own context; and do the reasoning and the final judgment itself.
- **Limits.** Usually one consultation per problem, and two at most per unit. If the advice says the plan itself is wrong, or two consultations haven't resolved the problem, go to 4.4.

Record each consultation in the log: unit, advisor model, request and advice file paths, and whether the advice worked. Tell the reviewer in 4.2 which advice the unit followed, so that a choice made on advice isn't flagged as unsanctioned.

**Escalating.** A Sonnet executor may report that a criterion can't be met, or come back with a result that misses the point of the unit. If the cause is the plan, that is 4.4. If a specific obstacle is the problem, consult an advisor first; that is cheaper than redoing the unit. If the unit as a whole turned out harder than rated, reclassify it as hard. Discard or reset the attempt (it is still uncommitted, or a single commit you can drop), and delegate it again to a fresh Opus executor (same agent type, `model: "opus"`). Give that agent the original brief plus the Sonnet report, so it doesn't repeat the same dead ends. Note the reclassification in the log.

Whoever executes the unit must:
- implement only that unit;
- follow the plan's constraints;
- run the unit's completion criteria and report the actual output;
- make exactly one commit following the repository's conventions (and any attribution instructions in effect).

Don't modify existing tests to make them pass; that changes the specification. If a criterion can't be met as written, report it rather than working around it.

**Check before review.** Re-run the key completion commands yourself, or at least inspect the reported output. Don't send a unit to review on the executor's word alone.

### 4.2 Adversarial review

After the unit is committed, launch **a fresh reviewer with effort medium** (`subagent_type: "ecoskills:reviewer"`) using `references/task-review-prompt.md`. Pick its model from the unit's tier: Sonnet for standard units, Opus for hard ones (`model: "sonnet"` or `"opus"`, `effort: "medium"`). Use the same tier even if you executed the unit yourself. On hard units, problems are subtle, and a reviewer weaker than the work tends to miss them or raise false findings. Give it:
- the plan path and the unit's plan section;
- the commit SHA;
- the verification output;
- the next unit's plan section.

Don't give it your own assessment. Do give it facts that put things out of scope, when they would otherwise invite false findings. For example: commit trailers follow the session's attribution instructions (a reviewer running on a different model tends to "correct" the model name). Also tell it about any command wrappers from Phase 3. It looks for:
- **contradictions with the plan**: deviations from decisions, constraints, or completion criteria, including scope creep;
- **problems for the next unit**: anything in this commit that would block or complicate the next unit, or that the next unit's premises no longer match.

A fresh context matters here: an executor's or orchestrator's context makes it easy to accept a deviation as reasonable.

### 4.3 Handle the findings

For each finding, decide whether it is valid. Reviewers can be wrong; check against the plan and the code. Reviewers sometimes can't write their findings file. In that case, take the findings from the final message and save them yourself, if they matter later (e.g. for the final review).

If the same false finding recurs across units, add a scope note for it to the remaining review prompts rather than rejecting it again each time.

- **Valid, local fix**: fix it yourself. If you delegated the unit, resume that agent with `SendMessage` (load it with `ToolSearch` if deferred), passing the review file path and the findings you accepted. The agent still has the unit's context. Amend the fix into the unit's commit, so that one unit stays one commit, and re-run the completion criteria.
- **Invalid**: note it in the log with the reason.
- **Serious**: see 4.4.

Re-review only if the fix was substantial, and at most once per unit. Then update the log and move to the next unit.

### 4.4 When to stop

Stop executing and consult the user if any of these appear:
- the plan cannot be carried out as written (a premise is false, a required option or tool doesn't exist, or a completion criterion is unreachable);
- a contradiction between the plan and reality, or within the plan, that affects later units;
- following the plan would violate one of its long-term guidelines, or would require changing one, or a condition the plan names for re-examining a decision has occurred;
- the remaining units' results would no longer meet the plan's goals.

Report what you found, with evidence (commands and output), which units are done (with SHAs), and which remaining units are affected. Then:
- If the plan needs revision rather than a local workaround, **propose a re-plan**. Write a self-contained re-plan request from `references/replan-prompt.md`: whoever revises the plan won't have this conversation, so the request carries the execution status, the evidence, and the decisions made so far. Show it to the user and ask how they want the plan revised: by handing the request to the planning process they normally use (a planning skill or workflow, or a separate session), or by revising it here together with you. Do nothing until the user decides. Revising a plan involves the user's judgment, so run the revision where the user can answer; if it runs in a subagent, relay its questions to the user.
- After a revised plan comes back, resume from Phase 1 for the remaining units. The revised plan replaces the committed one on the branch.

Minor deviations that don't affect later units don't warrant stopping. Record them in the log and in the commit message.

## Phase 5: Final review of the whole branch

When all units are done, launch **a fresh Opus reviewer with effort medium** (`subagent_type: "ecoskills:reviewer"`, `model: "opus"`, `effort: "medium"`) using `references/final-review-prompt.md`. It reviews the entire branch diff (`git diff <base-sha>...HEAD`) against the plan:
- whether the goals and success criteria are met;
- consistency across units;
- leftovers (debug code, temporary files, stale references);
- correctness and quality issues that per-unit reviews can miss.

Handle the findings as in 4.3. Fixes go in new commits, or are amended into the relevant unit's commit when they clearly belong there. Re-run the plan's overall verification afterwards. If the fixes were substantial, run the final review once more. If the review surfaces a serious problem, handle it as in 4.4.

## Phase 6: Finish

1. Remove the plan document in its own commit; it has served its purpose. Its decisions live on in the code and the commit history. If anything in it must outlive the branch (e.g. a maintenance policy the code points to), say so and ask the user where it should live before deleting.
2. Ask the user how to integrate: merge into which branch (default: the recorded base), open a PR, or leave the branch as is. Don't merge, push, or open a PR without that answer.
3. Carry out the choice. For a PR, write the description from the plan's goals and the unit list.
4. Clean up the worktree: use `ExitWorktree` if you entered with `EnterWorktree`, otherwise `git worktree remove <path>`. Delete the branch only if it was merged and the user agrees.
5. Report:
   - what was implemented (units and SHAs, or the PR URL);
   - deviations from the plan, and why;
   - review findings that were rejected, and why;
   - remaining manual steps the plan assigns to the user (e.g. checks on real hardware).
