<div align="center">

# plan-review

Five-agent objective-aligned plan review for AI coding agents.

English | [简体中文](./README.zh.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-6e47ff)](https://claude.ai/code)
[![Version](https://img.shields.io/badge/version-1.0.0-blue)](#)
[![GitHub stars](https://img.shields.io/github/stars/feariangod/karpathy-plan-review?style=social)](https://github.com/feariangod/karpathy-plan-review/stargazers)

</div>

---

`plan-review` is a reusable skill for reviewing and executing complex AI-agent plans before they turn into broken changes. The repository is named `karpathy-plan-review` for historical continuity; the skill name is now simplified to `plan-review`.

## Why This Exists

LLM-generated plans often fail for predictable reasons:

- The plan silently assumes facts that were never verified.
- The plan is reviewed by the same agent that produced it.
- Edge cases, references, IDs, and permission boundaries are discovered during execution instead of before execution.
- Verification says "looks right" instead of naming a command and expected result.

`plan-review` makes the task goal explicit first, then forces planning, review, supervision, and execution to stay aligned to that goal.

## What It Does

| Capability | What it enforces |
|---|---|
| Task total goal | Every agent works from the same explicit goal and success criteria |
| Shared task envelope | Goal, scope, evidence, round state, and boundaries are passed to every agent |
| Five-agent loop | Task Recognition, Supervisor, Plan, Review, and Execution agents have separate roles |
| P0/P1/P2 closure | Risks must be fixed or evidence-closed before convergence |
| Two clean rounds | Convergence requires two consecutive review rounds with zero new P0/P1/P2 |
| Permission stop gates | Execution stops at authorization, production access, compliance, money, contracts, missing data, or capability limits |

## Five-Agent Flow

```text
User input
  -> Task Recognition Agent: converts all received information into the task total goal
  -> Supervisor Agent: passes the shared task envelope and enforces role discipline
  -> Plan Agent: proposes the best goal-aligned plan
  -> Review Agent: finds P0/P1/P2 risks only
  -> Execution Agent: checks goal alignment, permission, and capability before execution
```

The core rule is simple: **the task total goal is the single source of truth**.

## Install

Install from GitHub in Claude Code:

```text
/plugin install feariangod/karpathy-plan-review
```

Manual Claude skill install:

```bash
mkdir -p ~/.claude/skills/plan-review
curl -L https://raw.githubusercontent.com/feariangod/karpathy-plan-review/main/SKILL.md \
  -o ~/.claude/skills/plan-review/SKILL.md
```

Manual Codex skill install:

```bash
mkdir -p ~/.codex/skills/plan-review
curl -L https://raw.githubusercontent.com/feariangod/karpathy-plan-review/main/SKILL.md \
  -o ~/.codex/skills/plan-review/SKILL.md
```

## When to Use

Use `plan-review` when:

- You are in plan mode or reviewing an implementation plan.
- A task spans multiple files, IDs, schemas, paths, references, or handoffs.
- The task has implicit requirements or unclear success criteria.
- Execution may hit permission, compliance, production, money, contract, missing-data, or capability boundaries.
- You want an independent risk review before spending tokens on execution.

Simple tasks can use compact outputs, but they still keep the task goal and review gates.

## Relationship to `karpathy-guidelines`

`karpathy-guidelines` is the coding discipline: think before coding, keep changes simple, edit surgically, and define verifiable success criteria.

`plan-review` applies the same Karpathy-style rigor to plans:

| | `karpathy-guidelines` | `plan-review` |
|---|---|---|
| Scope | Code and implementation behavior | Plans, handoffs, multi-step execution |
| Timing | While writing or reviewing code | Before executing complex plans |
| Mechanism | Four-principle discipline | Five-agent goal-aligned review loop |
| Verification | Tests and concrete checks | P0/P1/P2 closure plus grep-able verification |

Use both together for non-trivial engineering work.

## Legacy Name

Older prompts may refer to this workflow as `karpathy-plan-review`. The independent repository keeps that name in the URL. The current skill frontmatter uses:

```yaml
name: plan-review
```

New usage should prefer `plan-review`.

## License

MIT — see [LICENSE](./LICENSE).
