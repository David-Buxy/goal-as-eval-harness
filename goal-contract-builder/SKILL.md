---
name: goal-contract-builder
description: >-
  Use when aligning a user and agent on a broad task by turning the goal itself
  into a lightweight eval harness: clarify intent, state transition, completion
  semantics, trials, transcript capture, outcome probes, graders, iteration
  loop, blocked conditions, and stop rules. Use before long-running /goal work,
  vague project milestones, self-improving agent tasks, production-readiness
  checks, or any task where "done" must be proven rather than asserted.
---

# Goal Contract Builder

Build goals as eval harnesses.

A good goal is not just a task description. It tells the agent how to prove the work is done: what state should change, how attempts should be run, what trace must be captured, what outcome probes must pass, who or what grades the result, and when the agent should stop.

Splitting a large goal is optional. Split only when one goal cannot be evaluated honestly as a single harness.

## Core Idea

Every serious goal should answer:

- What does the user want to be different after this work?
- What does completion mean?
- What does completion not mean?
- What trials will the agent run?
- What transcript or trace will be preserved?
- What final outcome will be probed?
- What grader decides pass/fail or quality?
- When should the agent iterate, block, degrade, or stop?

## Codex `/goal` Length Limit

When the goal will be pasted into Codex `/goal`, the executable goal must stay under 4000 characters.

If the full harness is too long:

- Put only the current goal in the `/goal` text.
- Prefer concise bullets over long prose.
- Keep future goals as names only, or omit them.
- Move detailed rubrics, examples, and background into a linked/local artifact.
- Preserve the essentials: intent, state before/after, completion semantics, trials, transcript capture, graders, pass/blocked/stop conditions.
- Do not remove `completion_does_not_mean`; it prevents false completion.

## When To Use

Use this skill when the user wants to:

- Prepare a long-running `/goal` task.
- Turn a vague request into a verifiable goal.
- Decide whether a large task should be one goal or several.
- Force an agent to self-iterate instead of producing a quick report.
- Define completion criteria, evidence, and stop rules.
- Build a lightweight evaluation harness for an agent workflow.

Do not use this skill for tiny tasks where completion is already obvious.

For a full worked example, read `references/examples.md` only when a concrete pattern would help.

## Workflow

### 1. Align On The Intended Change

Restate the goal as a state transition:

```text
The user wants to move from [current state] to [desired state], so that [reason].
```

If the desired state is unclear, ask one to three short questions before writing the goal. Useful questions:

- What would make you feel this was actually done?
- What should not be claimed as solved when this finishes?
- Do you want diagnosis, risk reduction, capability recovery, an artifact, automation, or handoff?
- What evidence would you trust: tests, logs, files, database readback, URLs, screenshots, metrics, or human review?
- If the whole goal cannot be completed in one run, what partial state would still be valuable?

When a safe assumption is enough, state it instead of blocking.

Before the final goal, produce an Alignment Draft:

```markdown
## Alignment Draft
- Intended state change:
- Completion means:
- Completion does not mean:
- Split recommendation:
- Harness strength:
- Eval dataset needed:
- Risk confirmations needed:
- Assumptions:
- Questions:
```

Ask at most three questions when the draft has material uncertainty. If the user already gave enough context, continue with stated assumptions.

### 2. Decide Whether To Split

Do not split just to create a plan. First break down the problem, then split only when it improves evaluation.

Before deciding, produce a compact problem breakdown:

```yaml
problem_breakdown:
  subproblems:
    - id:
      question:
      why_it_matters:
      can_validate_independently:
      depends_on:
      risk_if_skipped:
  dependency_order:
  recommended_goal_units:
```

Use plain language. The purpose is to see what the big request contains, what must happen first, and which pieces are worth turning into goals.

Split when:

- The desired state contains independent state changes.
- Different parts require different graders or evidence.
- Some parts require unavailable credentials, user decisions, or external systems.
- One part is diagnostic but another requires capability recovery.
- A subproblem can be validated independently and skipping it would make later work hollow.
- A single goal would let the agent keep expanding without a clear stop rule.

Do not split when:

- One harness can clearly define task, trials, outcome probes, graders, and stop conditions.
- The subtasks are merely implementation steps inside one outcome.
- Splitting would create artificial progress without changing what the user cares about.

Good goal units usually satisfy:

- They create a clear state change.
- They can be independently checked.
- They unblock later work.
- They have their own evidence.
- They are small enough for one `/goal` run.

### 3. Choose Harness Strength

Pick the lightest harness that can honestly prove the goal.

- `light`: one real trial, artifact/outcome check, concise transcript. Use for one-off tasks with clear completion.
- `standard`: at least three trials or checks, transcript capture, basic programmatic or human grader. Use for reusable workflows, automation, or ambiguous tasks.
- `deep`: five or more iterations, explicit eval dataset, multiple graders, stronger stop rules. Use for self-improving capabilities, model/agent quality, retrieval, classification, generation, or production reliability.

Do not use `deep` to make simple tasks look sophisticated. Do not use `light` for a goal that claims a reusable capability improved.

### 4. Decide Whether An Eval Dataset Is Needed

Do not require an eval dataset for every goal. Require one when the goal claims reusable capability, quality improvement, model/agent performance, retrieval/classification/generation behavior, or production automation reliability.

Use this shape when needed:

```yaml
eval_dataset:
  required:
  reason:
  cases:
    - id:
      input:
      expected_behavior:
      scoring:
      required_evidence:
```

For one-off implementation or document tasks, use outcome probes and checklists instead of a dataset.

### 5. Check Risk Permissions

If the goal may perform risky operations, require explicit user confirmation or mark the goal blocked/degraded.

Risk confirmation overrides completeness. Even if the user provides a complete goal harness, do not execute risky operations until the user explicitly confirms them.

Common confirmations:

- `production_write`
- `external_send`
- `credential_use`
- `paid_api_or_cost`
- `delete_or_overwrite_data`
- `long_running_automation`
- `public_publish`

Record them as:

```yaml
requires_user_confirmation:
  - production_write
  - external_send
```

### 6. Write The Goal As A Harness

Use this schema unless a project has a stricter local format:

```yaml
id:
user_intent:
state_before:
state_after_expected:
completion_semantics:
  completion_means:
  completion_does_not_mean:

split_decision:
  split_needed:
  reason:
  parent_goal:

problem_breakdown:
  subproblems:
  dependency_order:
  recommended_goal_units:

harness_strength:

requires_user_confirmation:

eval_dataset:
  required:
  reason:
  cases:

task:
  objective:
  scope:
  out_of_scope:
  inputs:
  allowed_actions:
  constraints:

trial_plan:
  minimum_trials:
  maximum_trials:
  trial_inputs:
  retry_policy:
  allowed_strategy_changes:

transcript_capture:
  record:
    - commands_or_tools
    - inputs_or_queries
    - returned_results
    - matched_results
    - rejected_results
    - errors
    - adjustments

outcome_probe:
  checks:
    - artifact_exists
    - schema_valid
    - tests_pass
    - links_or_records_verifiable
    - state_change_observed

graders:
  programmatic:
  llm_judge:
  human_review:

iteration_loop:
  minimum_iterations:
  maximum_iterations:
  per_iteration_must_include:
    - hypothesis
    - action
    - result
    - lesson
    - next_adjustment
  early_stop_allowed_if:

pass_conditions:
blocked_conditions:
degraded_conditions:
stop_conditions:
handoff:
```

### 7. Make Completion Semantics Explicit

Always include both sides:

```yaml
completion_semantics:
  completion_means:
    - What the user may legitimately believe after the goal passes.
  completion_does_not_mean:
    - What remains unsolved or unproven.
```

This prevents diagnostic work from being mistaken for capability recovery, and prevents a report from being mistaken for a working system.

### 8. Require Real Trials For Capability Goals

If the user wants a capability, not just an artifact, include a real trial plan.

For self-iterating agent work:

- Set `minimum_trials` or `minimum_iterations`.
- Require each iteration to record hypothesis, action, result, lesson, and next adjustment.
- Forbid early stop unless the pass conditions are already met.
- Require at least one strategy adjustment when a trial fails or returns low-quality results.

Fast completion is acceptable only when the harness passes, not when the agent merely produced a document.

### 9. Capture The Transcript

Do not grade only the final answer. Require trace evidence when the path matters:

- commands or tool calls
- queries and parameters
- returned/matched/rejected counts
- error types
- files or records changed
- retries and strategy changes
- reasons for blocking or degrading

The transcript does not need to be verbose, but it must be enough for another human or agent to verify the work.

### 10. Choose Graders

Use the narrowest reliable grader:

- `programmatic`: tests, schema checks, file existence, URL validation, database readback, deterministic scripts.
- `llm_judge`: semantic relevance, qualitative summaries, rubric scoring.
- `human_review`: business judgment, subjective quality, safety-sensitive or high-stakes decisions.

Prefer programmatic checks for structure and state. Use LLM or human review for meaning and quality.

### 11. Fix Misaligned Goals

Rewrite, question, or split goals that contain:

- No state transition.
- No `completion_does_not_mean`.
- No real trial for a capability claim.
- No transcript capture.
- No grader.
- No interaction point when the user's completion semantics are unclear.
- No eval dataset for a claimed reusable capability.
- No risk confirmation for production writes, external sends, credential use, destructive actions, public publishing, or long-running automation. Risk confirmation is required even when the rest of the goal is fully specified.
- No stop condition.
- A hidden dependency on credentials, private data, external access, or human taste.
- A claim that explains a problem but sounds like the problem was fixed.
- A report-only artifact for a user who asked for a working capability.

## Output Template

~~~markdown
# Goal Harness

## Alignment Brief
- User intent:
- Current state:
- Desired state:
- Completion means:
- Completion does not mean:
- Problem breakdown:
- Split decision:
- Harness strength:
- Eval dataset needed:
- Risk confirmations needed:

## Recommended Goal

```yaml
id:
user_intent:
state_before:
state_after_expected:
completion_semantics:
  completion_means:
  completion_does_not_mean:

split_decision:
  split_needed:
  reason:
  parent_goal:

problem_breakdown:
  subproblems:
  dependency_order:
  recommended_goal_units:

harness_strength:

requires_user_confirmation:

eval_dataset:
  required:
  reason:
  cases:

task:
  objective:
  scope:
  out_of_scope:
  inputs:
  allowed_actions:
  constraints:

trial_plan:
  minimum_trials:
  maximum_trials:
  trial_inputs:
  retry_policy:
  allowed_strategy_changes:

transcript_capture:
  record:

outcome_probe:
  checks:

graders:
  programmatic:
  llm_judge:
  human_review:

iteration_loop:
  minimum_iterations:
  maximum_iterations:
  per_iteration_must_include:
  early_stop_allowed_if:

pass_conditions:
blocked_conditions:
degraded_conditions:
stop_conditions:
handoff:
```

## Future Goals
1.
2.
3.
~~~

## Full Example

Read `references/examples.md` when the user asks for an example goal or when a concrete pattern would improve the generated contract.

## Common Failure Modes

- Goal-as-task: the goal says what to do but not how to prove it.
- Report-as-capability: a report is produced but no workflow can be rerun.
- No trial plan: the agent can stop after one shallow attempt.
- No transcript: nobody can inspect the path taken.
- No grader: completion depends on the agent's self-report.
- Completion ambiguity: diagnosis is mistaken for repair.
- Source failure confusion: a blocked channel is treated as a negative result.
- Fake split: many small goals are created without improving evaluability.
- Endless refinement: no stop condition bounds the work.
