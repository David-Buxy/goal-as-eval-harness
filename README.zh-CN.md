# Goal as Eval Harness

`goal-contract-builder` 是一个 Codex skill，用来把目标写成轻量级的 eval harness。

核心思想：

> Build goals as eval harnesses.

也就是：

> 一个好的目标，不只是告诉 agent 要做什么，还要规定 agent 如何证明自己真的做成了。

## 它解决什么问题

很多 agent 任务失败，不是因为 agent 完全不会做，而是因为目标写得太像一句愿望：

```text
帮我把这个能力做好。
```

这种目标很容易导致 agent 很快产出一份报告，然后宣布完成。用户真正想要的可能是一个可复跑的能力、一个经过多轮验证的流程，或者一个能稳定判断成功/失败的闭环。

这个 skill 的作用，是把目标改写成一个可以运行、观察、判分和停止的 harness。

## 一个 Goal 应该内置什么

一个可用的 goal 应该说明：

- 用户希望什么状态发生变化
- 完成意味着什么
- 完成不意味着什么
- agent 至少要跑哪些 trial
- 过程 transcript 要记录什么
- 最终 outcome 要怎么探测
- 用什么 grader 判断通过、失败或降级
- 什么情况下继续迭代
- 什么情况下标记 blocked / degraded
- 什么情况下停止

换句话说，goal 本身就是 agent 工作的评测闭环。

## 什么时候使用

适合在这些场景使用：

- 准备启动一个长期 `/goal` 任务
- 任务比较模糊，完成标准不清楚
- 希望 agent 不要只写报告，而是多轮试错和自我迭代
- 想把一个大目标拆成可分别验收的小目标
- 要求 agent 留下过程证据、运行记录、失败原因和停止条件
- 要做生产准备、能力验证、自动化闭环或复杂项目交接

不太需要使用的场景：

- 改一行文案
- 运行一个明确命令
- 回答一个简单问题
- 修一个完成标准非常明确的小 bug

## 安装

把 skill 目录复制到 Codex skills 目录：

```bash
mkdir -p ~/.codex/skills
cp -R goal-contract-builder ~/.codex/skills/
```

然后可以显式调用：

```text
$goal-contract-builder
请把这个任务写成 goal-as-eval-harness：……
```

也可以自然语言描述：

```text
请把这个任务整理成可交给 /goal 长跑的目标，要求包含 trial、transcript、outcome probe、grader、pass conditions、blocked conditions 和 stop rules。
```

## 示例

普通目标：

```text
建立一个关键词讨论检索能力。
```

用这个 skill 改写后，会变成类似：

```text
输入一个关键词，至少尝试三个公开来源；
每轮记录 query、returned、matched、rejected、errors；
根据失败结果调整检索策略；
最终生成 JSON 和 Markdown；
用 schema checker 验证结构；
用人工或 LLM judge 抽查 Top 结果相关性；
达到通过条件或 blocked 条件后停止。
```

这样 agent 就不能只搜索一次、写一份总结然后说完成。它必须证明自己真的跑过、失败过、调整过，并留下可复核证据。

## Codex `/goal` 字数预算

如果要把 goal 粘贴到 Codex `/goal` 里，建议把可执行 goal 控制在 4000 中文字以内。

如果完整 harness 很长：

- `/goal` 里只放当前最重要的 goal
- 详细 rubric、背景材料和 future goals 放到旁路文档
- 保留最关键内容：意图、状态变化、完成语义、trial、transcript、grader、pass / blocked / stop 条件
- 不要删掉“完成不意味着什么”，这是防止虚假完成的关键

## 仓库结构

```text
goal-as-eval-harness/
├── README.md
├── README.zh-CN.md
├── LICENSE
└── goal-contract-builder/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

## 许可证

MIT
