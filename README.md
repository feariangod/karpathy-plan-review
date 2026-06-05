# karpathy-plan-review

Multi-Agent Ping-Pong Review & Execution Loop — a [Claude Code](https://claude.ai/code) skill for plan review and execution.

Generator and reviewer subagents iterate independently, exchanging reviews through a structured interface, converging round by round until the plan is solid — then atomically executing with grep-hardened verification.

## How it works

```
User task
  ↓
Phase 0: Explore — read all target files, no memory
  ↓
Phase 1: Ping-Pong Loop
  ├─ Generator subagent (3-alternative selection → plan)
  ├─ Reviewer subagent (Karpathy 4 principles + completeness → P0/P1/P2)
  ├─ Reviewer decides if another round is needed
  ├─ P0/P1 remain? → Generator receives report → improves → re-review
  └─ Converged (2 consecutive rounds zero new P0/P1) → Phase 2
  ↓
Phase 2: Atomic execution — grep verification at each step
  ↓
Phase 3: Closure — knowledge distillation
```

## Key features

- **Independent agent perspectives** — generator and reviewer are separate subagents, eliminating self-review blind spots
- **Karpathy's 4 principles** applied to plans (not just code): Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution
- **Hard convergence criteria** — 2 consecutive rounds with zero new P0/P1 findings
- **Grep-verifiable validation** — all success criteria must be mechanically checkable
- **Max 5 rounds** — force-stop with human judgment flag if no convergence
- **"Do not change" list** — explicitly tracked, preventing repeated findings

## Installation

Install via Claude Code's plugin system:

```
/plugin install feariangod/karpathy-plan-review
```

Or add to `.claude/settings.json`:

```json
{
  "plugins": [
    {
      "type": "github",
      "repo": "feariangod/karpathy-plan-review"
    }
  ]
}
```

## Triggers

- Plan mode activation (`EnterPlanMode`)
- User writes a plan file
- "Review this plan" / "Karpathy review"
- Any multi-step complex task (self-determined)

## Relationship with karpathy-guidelines

| | karpathy-guidelines | karpathy-plan-review |
|---|---|---|
| Scope | Code (single line/function/file) | Plans/designs (multi-step/multi-file) |
| Timing | While coding | After planning, before execution |
| Mechanism | 4-principle behavioral guidance | Multi-agent ping-pong + convergence + grep verify |
| Verification | Test-driven (write test → make pass) | grep-hardened (grep/awk/wc checkable) |

They are complementary, not alternatives. `karpathy-guidelines` is the reviewer subagent's core framework within this skill.

## License

MIT
