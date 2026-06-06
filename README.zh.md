<div align="center">

# plan-review

面向 AI 编码 Agent 的五 Agent 目标对齐计划审查 skill。

[English](./README.md) | 简体中文

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-6e47ff)](https://claude.ai/code)
[![Version](https://img.shields.io/badge/version-1.0.0-blue)](#)
[![GitHub stars](https://img.shields.io/github/stars/feariangod/karpathy-plan-review?style=social)](https://github.com/feariangod/karpathy-plan-review/stargazers)

</div>

---

`plan-review` 是一个可复用 skill，用于在复杂 AI Agent 计划进入执行前做目标对齐审查。仓库名保留为 `karpathy-plan-review` 是为了延续历史命名；当前 skill 名已经简化为 `plan-review`。

## 为什么需要它

LLM 生成的计划经常因为类似原因失败：

- 计划默默假设了一些没有验证过的事实。
- 产出计划的 Agent 又自己审查自己的计划。
- 边界场景、引用、ID、权限边界在执行中才暴露。
- 验证标准写成“看起来正确”，而不是命令和预期结果。

`plan-review` 先把任务总目标明确下来，再强制方案、审查、监督和执行都围绕这个目标运行。

## 它做什么

| 能力 | 强制约束 |
|---|---|
| 任务总目标 | 每个 Agent 都围绕同一个显式目标和成功标准工作 |
| 共享任务信封 | 把目标、范围、证据、轮次状态和边界传给每个 Agent |
| 五 Agent 闭环 | 任务识别、监督、方案、审查、执行职责分离 |
| P0/P1/P2 闭合 | 风险必须被修复，或用证据闭合后才能收敛 |
| 连续两轮干净 | 收敛要求连续两轮审查零新增 P0/P1/P2 |
| 权限停机门 | 遇到授权、生产权限、合规、资金、合同、缺资料或能力边界时停止 |

## 五 Agent 流程

```text
用户输入
  -> Task Recognition Agent / 任务识别 Agent：把所有信息转成任务总目标
  -> Supervisor Agent / 监督 Agent：传递共享任务信封并监督角色边界
  -> Plan Agent / 方案 Agent：产出与任务总目标对齐的最优方案
  -> Review Agent / 审查 Agent：只发现 P0/P1/P2 风险
  -> Execution Agent / 执行 Agent：执行前检查目标对齐、权限和能力边界
```

核心规则很简单：**任务总目标是唯一权威来源**。

## 安装

在 Claude Code 中从 GitHub 安装：

```text
/plugin install feariangod/karpathy-plan-review
```

手动安装到 Claude：

```bash
mkdir -p ~/.claude/skills/plan-review
curl -L https://raw.githubusercontent.com/feariangod/karpathy-plan-review/main/SKILL.md \
  -o ~/.claude/skills/plan-review/SKILL.md
```

手动安装到 Codex：

```bash
mkdir -p ~/.codex/skills/plan-review
curl -L https://raw.githubusercontent.com/feariangod/karpathy-plan-review/main/SKILL.md \
  -o ~/.codex/skills/plan-review/SKILL.md
```

## 什么时候使用

适合在这些场景使用 `plan-review`：

- 进入计划模式，或需要审查实施计划。
- 任务涉及多个文件、ID、schema、路径、引用或 handoff。
- 任务存在隐含需求或成功标准不清晰。
- 执行可能触及权限、合规、生产系统、资金、合同、缺资料或能力边界。
- 想在执行前用独立审查找出计划风险。

简单任务可以用紧凑输出，但仍然保留任务目标和审查门禁。

## 与 `karpathy-guidelines` 的关系

`karpathy-guidelines` 是编码纪律：先思考再编码、保持简洁、精准修改、定义可验证成功标准。

`plan-review` 把同样的 Karpathy 风格严谨性应用到计划层：

| | `karpathy-guidelines` | `plan-review` |
|---|---|---|
| 范围 | 代码与实现行为 | 计划、handoff、多步骤执行 |
| 时机 | 写代码或审查代码时 | 复杂计划执行前 |
| 机制 | 四原则行为纪律 | 五 Agent 目标对齐审查闭环 |
| 验证 | 测试与具体检查 | P0/P1/P2 闭合 + grep 可验证条件 |

非琐碎工程任务建议两者配合使用。

## 旧名称

旧提示词可能仍称这个工作流为 `karpathy-plan-review`。独立仓库 URL 也保留这个名字。当前 skill frontmatter 使用：

```yaml
name: plan-review
```

新用法优先使用 `plan-review`。

## 许可

MIT — 见 [LICENSE](./LICENSE)。
