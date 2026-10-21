---
name: orchestrated-execution
description: Executes a structured plan document (typically one produced by the orchestrated-planning skill under docs/plans/) task by task in a dedicated branch and git worktree — clarifying open points with the user first, splitting the plan into commit-sized units, delegating tasks to Sonnet agents when context isolation helps, putting every task through adversarial review by a fresh Sonnet agent, stopping to consult the user (and proposing a re-plan) when a serious contradiction surfaces, getting a final Opus review of the whole branch, and finally removing the plan document and merging or opening a PR. Use this whenever the user asks to implement, execute, or carry out a plan document, or says things like 「計画を実行して」「この計画書どおりに実装して」「docs/plans の計画を進めて」, even if they don't name this skill. Do not use for ad-hoc changes with no plan document.
---

# Orchestrated Execution

Carry out a plan document faithfully, one commit-sized task at a time, with an independent review after every task and a review of the whole at the end. The plan is the specification. The job is to implement it, not to redesign it. When the plan turns out to be wrong in a way that matters, stop and bring it back to the user instead of improvising.

## Overview

```
Phase 1  Analyze the plan and confirm open points   (this session) → user answers
Phase 2  Split into ordered, commit-sized units     (this session)
Phase 3  Branch + worktree                          (this session)
Phase 4  Execute each unit                          (this session or a Sonnet agent)
           └ per unit: verify → commit → adversarial review (fresh Sonnet, effort medium) → fix
           └ serious problem → stop, consult the user, propose a re-plan
Phase 5  Final review of the whole branch           (fresh Opus, effort medium) → fix
Phase 6  Remove the plan, merge or open a PR, clean up the worktree (each step confirmed with the user)
```

Keep an **execution log** in the scratchpad directory: one row per unit with its status, commit SHA, executor (self or agent ID), review file, and outcome. Save every delegation brief and review result next to it as files. This lets work resume after the conversation context is summarized, and lets agents read artifacts by path.

---

## Phase 1: Analyze the plan and confirm open points

Read the whole plan document before touching anything. Then check it from the implementer's position:

- **Unresolved items and defaults**: list every item the plan marks as unresolved, owner-to-decide, vetoable, or conditional, together with its stated default.
- **Ambiguities**: places where two reasonable implementers would do different things, wording like "as appropriate", and missing completion criteria.
- **Drift**: spot-check the plan's "background context" against the repository as it is now (key files, line references, the verification baseline). Plans go stale. A changed premise is cheaper to catch now than in task 5.
- **Preconditions**: tools, credentials, or environments the verification commands need, and whether they are available here.
- **Internal consistency**: tasks that contradict decisions, and dependencies that don't match the stated order.

Ask the user about the points whose answers change what gets built. Use concrete options, and put the plan's default first. Don't ask about things the plan has already decided; the plan is the agreed specification.

Record the answers **in the plan document itself**: update the unresolved-items table, and strike or mark vetoed items. Delegated agents and reviewers read the plan, not this conversation, so the plan must reflect the decisions.

## Phase 2: Split into ordered units

Turn the plan's tasks into an ordered list of **units**. Each unit is one feature-level change that belongs in a single commit and can be verified on its own.

- Usually the plan's tasks map one-to-one. Split a task that mixes independent changes, and merge tasks that cannot be verified separately.
- Respect the plan's execution order and dependencies. Drop units for vetoed items, and conditional units whose condition is unmet.
- For each unit, note the plan section it implements, its completion criteria, and whether you will execute it yourself or delegate it (see Phase 4).

Show the user the unit list briefly and proceed, unless the split departs from the plan in a way the user should weigh in on.

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

**Doing it yourself vs. delegating to a Sonnet agent.** Delegate (`Agent` with `model: "sonnet"`, `subagent_type: "general-purpose"`) when context isolation pays off:
- the unit is self-contained and the plan specifies it precisely (exact snippets, exact commands);
- or it requires reading many files, or produces long command output that would crowd this session's context.

Do it yourself when the unit is small, needs judgment that spans several units, or touches something that just changed because of a review.

When delegating, write the brief from `references/delegation-brief.md` to a file and point the agent at it. Record the agent ID in the log; you may need to resume it in 4.3.

Whoever executes the unit must:
- implement only that unit;
- follow the plan's constraints;
- run the unit's completion criteria and report the actual output;
- make exactly one commit following the repository's conventions (and any attribution instructions in effect).

Don't modify existing tests to make them pass; that changes the specification. If a criterion can't be met as written, report it rather than working around it.

**Check before review.** Re-run the key completion commands yourself, or at least inspect the reported output. Don't send a unit to review on the executor's word alone.

### 4.2 Adversarial review

After the unit is committed, launch **a fresh Sonnet agent with effort medium** (`model: "sonnet"`, `effort: "medium"`) using `references/task-review-prompt.md`. Give it:
- the plan path and the unit's plan section;
- the commit SHA;
- the verification output;
- the next unit's plan section.

Don't give it your own assessment. Do give it facts that put things out of scope, when they would otherwise invite false findings. For example: commit trailers follow the session's attribution instructions (a reviewer running on a different model tends to "correct" the model name). Also tell it about any command wrappers from Phase 3. It looks for:
- **contradictions with the plan**: deviations from decisions, constraints, or completion criteria, including scope creep;
- **problems for the next unit**: anything in this commit that would block or complicate the next unit, or that the next unit's premises no longer match.

A fresh context matters here for the same reason as in planning: an executor's or orchestrator's context makes it easy to accept a deviation as reasonable.

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
- following the plan would violate one of its long-term guidelines, or would require changing one;
- the remaining units' results would no longer meet the plan's goals.

Report what you found, with evidence (commands and output), which units are done (with SHAs), and which remaining units are affected. Then:
- If the plan needs revision rather than a local workaround, **propose re-planning with the `orchestrated-planning` skill**. Gather the information the re-plan needs and write a prompt from `references/replan-prompt.md`, then show it to the user and ask whether to run it. Run it only on the user's go-ahead. Planning involves user approval of its task assignment, so the re-plan should run where the user can answer (this session, invoking the skill with the prompt). Use a subagent only if the user prefers it and the subagent can spawn agents. In that case, relay its approval questions to the user.
- After a revised plan comes back, resume from Phase 1 for the remaining units. The revised plan replaces the committed one on the branch.

Minor deviations that don't affect later units don't warrant stopping. Record them in the log and in the commit message.

## Phase 5: Final review of the whole branch

When all units are done, launch **a fresh Opus agent with effort medium** (`model: "opus"`, `effort: "medium"`) using `references/final-review-prompt.md`. It reviews the entire branch diff (`git diff <base-sha>...HEAD`) against the plan:
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
