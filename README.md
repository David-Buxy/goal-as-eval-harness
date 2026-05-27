# Goal as Eval Harness

`goal-contract-builder` is a Codex skill for writing goals as lightweight eval harnesses.

中文说明见：[README.zh-CN.md](README.zh-CN.md)

The central idea:

> Build goals as eval harnesses.

A good goal does more than tell an agent what to do. It tells the agent how to prove the work is done:

- what state should change
- what completion means
- what completion does not mean
- what trials the agent should run
- what transcript or trace should be preserved
- what outcome probes must pass
- what grader decides quality or pass/fail
- when the agent should iterate, block, degrade, or stop

This is meant for long-running agent tasks, self-improving workflows, vague project milestones, production-readiness checks, and any work where "done" must be proven rather than asserted.

## Why This Exists

Agents often complete vague goals too quickly because the goal is only a task description:

```text
Find discussions about this keyword.
```

A goal-as-harness turns that into something evaluable:

```text
Try at least three sources, record returned/matched/rejected/errors, adjust strategy after failures, produce structured outputs, run schema checks, review top matches, and stop only when pass conditions or blocked conditions are met.
```

This pattern is inspired by agent evaluation ideas such as task, trial, transcript, outcome, grader, and harness. The skill adapts those ideas to everyday `/goal` work.

## Repository Structure

```text
goal-as-eval-harness/
├── README.md
├── LICENSE
└── goal-contract-builder/
    ├── SKILL.md
    ├── references/
    │   └── examples.md
    └── agents/
        └── openai.yaml
```

## Install

Copy the skill folder into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
cp -R goal-contract-builder ~/.codex/skills/
```

Then invoke it explicitly:

```text
$goal-contract-builder
Turn this task into a goal-as-eval-harness: ...
```

Or ask naturally for a verifiable `/goal` contract with trials, transcript capture, outcome probes, graders, and stop rules.

## Example Prompt

```text
$goal-contract-builder
I want an agent to build a reusable keyword discussion retrieval capability.
Do not let it stop after one search. Make the goal itself an eval harness with multiple trials, transcript capture, outcome probes, graders, pass conditions, blocked conditions, and stop rules.
```

## License

MIT
