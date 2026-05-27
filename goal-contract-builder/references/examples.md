# Goal Contract Examples

Load this file only when a concrete example would help write or audit a goal contract.

## Keyword Discussion Retrieval

```yaml
id: keyword-discussion-retrieval-harness
user_intent: >
  The user wants a reusable capability: given a keyword, find relevant public
  discussions and distinguish channel failure from lack of discussion.
state_before: >
  The agent may be blocked by a single failed source and may report a closeout
  instead of building the broader retrieval capability.
state_after_expected: >
  A repeatable keyword discussion retrieval workflow exists. It tries multiple
  public sources, records each source status, filters ambiguity, and outputs
  reviewable JSON/Markdown results.
completion_semantics:
  completion_means:
    - The keyword retrieval workflow has been tried across multiple sources.
    - Results and failures are traceable.
    - A failed source no longer blocks the whole capability.
  completion_does_not_mean:
    - Every source is fixed or stable.
    - The results are exhaustive.
    - The results are ready for production writes without review.

split_decision:
  split_needed: false
  reason: >
    The immediate need is one capability harness. Source-specific recovery can
    be split later if a source remains blocked.
  parent_goal: public discussion discovery

problem_breakdown:
  subproblems:
    - id: source_access
      question: Which public sources can be queried from the current environment?
      why_it_matters: Failed sources must not be confused with no discussion.
      can_validate_independently: true
      depends_on: []
      risk_if_skipped: A single blocked source can falsely block the whole capability.
    - id: relevance_filtering
      question: Can the workflow separate intended keyword meaning from unrelated uses?
      why_it_matters: Ambiguous keywords can produce misleading matches.
      can_validate_independently: true
      depends_on:
        - source_access
      risk_if_skipped: The workflow may return noisy results as matched.
    - id: output_artifacts
      question: Are results and failures written to reusable artifacts?
      why_it_matters: The capability must be rerunnable and reviewable.
      can_validate_independently: true
      depends_on:
        - source_access
        - relevance_filtering
      risk_if_skipped: The agent may only produce an unreproducible report.
  dependency_order:
    - source_access
    - relevance_filtering
    - output_artifacts
  recommended_goal_units:
    - Keep these together for the first harness because they jointly prove the minimum retrieval capability.

harness_strength: deep

requires_user_confirmation: []

eval_dataset:
  required: true
  reason: >
    This goal claims a reusable retrieval capability, so one keyword is not
    enough to prove general behavior.
  cases:
    - id: primary_keyword
      input: example keyword
      expected_behavior: Find relevant public discussions for the intended meaning of the keyword.
      scoring: Top results include stable references and relevance reasons.
      required_evidence: JSON/Markdown output with source status and URLs.
    - id: ambiguous_keyword
      input: ambiguous keyword
      expected_behavior: Separate intended meaning from unrelated uses of the same term.
      scoring: Rejected results include disambiguation reasons.
      required_evidence: rejected/candidate records with reasons.
    - id: source_failure
      input: primary keyword with one source unavailable
      expected_behavior: Mark the source as channel_failed and continue with fallback sources.
      scoring: channel_failed is not reported as no_discussion_found.
      required_evidence: transcript records error type and fallback attempt.

task:
  objective: >
    Build and validate a keyword-to-discussion retrieval workflow using one sample keyword.
  scope:
    - Try at least three public source types.
    - Generate structured JSON and human-readable Markdown.
    - Mark each result as matched, candidate, rejected, or channel_failed.
  out_of_scope:
    - Do not write to production systems.
    - Do not require private credentials.
    - Do not claim exhaustive coverage.
  inputs:
    keyword: example keyword
    disambiguation_terms:
      - product name
      - company name
      - use case
  allowed_actions:
    - Run public search/API probes.
    - Add or revise a local script.
    - Adjust queries and filters based on results.
  constraints:
    - Do not treat source errors as no discussion found.
    - Do not force low-relevance results into matched.

trial_plan:
  minimum_trials: 3
  maximum_trials: 8
  trial_inputs:
    - baseline keyword query
    - keyword plus disambiguation query
    - source-specific query variants
  retry_policy: >
    Retry a failed source at most once unless the query or access path changes.
  allowed_strategy_changes:
    - query rewrite
    - source fallback
    - relevance filter adjustment

transcript_capture:
  record:
    - source tried
    - query used
    - returned count
    - matched count
    - rejected count
    - errors
    - lesson
    - next adjustment

outcome_probe:
  checks:
    - output JSON exists and is valid
    - output Markdown exists
    - at least three source statuses are recorded
    - matched results include URLs or stable references
    - channel_failed is distinct from no_match

graders:
  programmatic:
    - validate output schema
    - count sources and statuses
  llm_judge:
    - judge whether top matched results are semantically relevant to the keyword intent
  human_review:
    - review top results before using them as evidence

iteration_loop:
  minimum_iterations: 3
  maximum_iterations: 8
  per_iteration_must_include:
    - hypothesis
    - action
    - result
    - lesson
    - next_adjustment
  early_stop_allowed_if:
    - at least three sources were tried
    - at least five relevant results passed review
    - output artifacts and transcript are complete

pass_conditions:
  - Minimum iterations are complete or early-stop conditions are met.
  - Outputs are valid and reviewable.
  - Source failures are explained without blocking the whole workflow.
blocked_conditions:
  - No public source can be accessed from the environment.
  - The user requires a private source but credentials are unavailable.
degraded_conditions:
  - Some sources fail but at least one fallback works.
  - Only snippets are available, not full content.
stop_conditions:
  - Pass conditions are met.
  - Maximum iterations are reached with a clear best result and next steps.
  - All sources are blocked with evidence.
handoff:
  - If relevance is weak, create a follow-up goal for classifier/filter improvement.
  - If a specific source is important but blocked, create a source recovery goal.
```
