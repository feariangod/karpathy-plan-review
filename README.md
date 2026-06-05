# karpathy-plan-review

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-6e47ff)](https://claude.ai/code)
[![Version](https://img.shields.io/badge/version-0.1.0-blue)]()

**LLMs write plans. Plans have blind spots. Blind spots become bugs. This skill makes sure they don't.**

</div>

---

## The Problem

You ask Claude Code to implement a multi-file change. It writes a plan, then executes it. But here is what happens:

- The plan assumes a file exists. It does not.
- The plan generates a duplicate ID. Nothing catches it.
- The plan says "update all references." It misses three.
- The plan's verification step is "visually confirm the output looks correct."

**The agent reviews its own plan.** This is the core design premise of this skill: a single agent has inherent difficulty finding the holes in its own thinking -- the same blind spots that produced the plan are the ones reviewing it. The result: plans that feel solid but break at step three, wasted tokens on rollbacks, and a creeping distrust of LLM-generated execution plans.

## The Solution

**Two agents. Alternating rounds. Hard convergence criteria. Zero self-review.**

`karpathy-plan-review` turns plan review into a Ping-Pong loop between two independent subagents:

| Role | Responsibility | Constraint |
|------|---------------|------------|
| **Generator** | Produces the plan. Brainstorms >=3 alternatives per decision. Responds to review feedback. | Never reviews its own output. |
| **Reviewer** | Audits the plan against Karpathy's 4 principles. Finds P0/P1/P2 issues. Decides if another round is needed. | Never proposes solutions. Only finds problems. |

They alternate until **2 consecutive rounds produce zero new P0/P1 findings**. Then -- and only then -- the plan enters atomic execution with grep-hardened verification at every step.

The host agent never touches plan content. It is a scheduler, not a participant. This is what eliminates the self-review blind spot.

## Before vs After

Here is an illustrative example: a task to add a `status` column to a markdown table used as a cross-file reference registry.

### Before (single-agent plan -> execute)

Plan: add column, update references, verify manually. Execution: column added but 3 rows have wrong column counts; grep finds 12 references but 9 are missed (file paths changed since last index); verification is "looks correct" (it is not). **Outcome**: broken table, dangling references, multiple debugging rounds.

### After (Ping-Pong review -> converge -> execute)

**Round 1** Reviewer finds: no column-count pre-check, no `grep -rl` pre-scan mandated, verification is "visually confirm" (not grep-able). **Round 2** Generator adds pre-check commands and grep-able verification. **Round 3** Reviewer finds zero new P0/P1 (second consecutive round). **Converged.** Execution proceeds with `awk` column validation, grep-verified reference updates, and audit trail saved.

The difference is not "more careful thinking." It is a process that makes oversights mechanically impossible to ship.

## How It Works

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
Phase 3: Closure — knowledge distillation (optional, requires companion skill)
```

**Hard limits:**
- Max 5 rounds. If no convergence by round 5, force-stop with a human-judgment flag. Execution is blocked.
- Reviewer produces the same P0 twice in a row? The generator is not responding to feedback. Host agent intervenes -- no infinite loops.
- Reviewer declares a task simple enough for one round? One round it is. (Reviewer decides, not generator -- eliminates the self-policing paradox.)

## Karpathy's 4 Principles, Operationalized

This skill applies Andrej Karpathy's coding principles to **plans** (not just code), and makes them mechanically enforceable:

| Principle | Applied to Code | Applied to Plans |
|-----------|----------------|-----------------|
| **Think Before Coding** | Understand the problem before typing. | Every assumption must be explicit, with a file path and line number. Unverifiable assumptions are risks. |
| **Simplicity First** | The simplest solution that works. | No "just in case" steps. Every change must trace to a user requirement. Cut what is not needed. |
| **Surgical Changes** | Change only what must change. | Every edit must cite the decision that drives it. Untouched lines between edits prove surgical precision. |
| **Goal-Driven Execution** | Verify, do not assume. | Every verification must be a `grep`, `awk`, or `wc` command with a defined expected output. "Looks correct" is a P1 finding. |

The reviewer subagent uses `karpathy-guidelines` as its core framework. This skill is that framework wrapped in a multi-agent process that makes it impossible to skip.

## Key Features

- **Independent agent perspectives** -- generator and reviewer are separate subagents. No agent reviews its own output.
- **Hard convergence criteria** -- 2 consecutive rounds with zero new P0/P1. Mechanically defined, not "feels good enough."
- **Grep-verifiable validation** -- all success criteria are shell commands with expected output. Verification output saved as audit trail. (Assumes a Unix-like environment with `grep`, `awk`, and `wc` available -- these are standard on macOS and Linux, the two platforms Claude Code supports.)
- **Max 5 rounds** -- force-stop with human judgment flag if no convergence. No infinite loops.
- **"Do Not Change" list** -- items the Generator has explicitly decided not to modify, each with a reason. Before each review round, the Reviewer reads this list and skips already-acknowledged items, preventing the same issue from being re-raised every round and blocking convergence. The Generator may add to this list when it accepts a Reviewer finding as a known trade-off.
- **Assumption surfacing** -- every round, the Generator produces a current assumption list. Each assumption names what is assumed, why, and what would break if it is wrong. "No assumptions" is not a valid answer -- even "the file at this path is a valid YAML document" counts.
- **Evidence strength grading** -- every finding and claim is tagged High (verified against actual file contents or command output), Medium (logical inference from known patterns), or Low (speculation / educated guess). No fabricated certainty -- if confidence is Low, the item is flagged for human judgment rather than silently treated as fact.
- **Knowledge closure** -- after execution, optionally distills decisions, do-not-change items, and verification results into structured knowledge assets (requires the companion `closure-knowledge-distillation` skill, see Dependencies).

## Dependencies

### Core Dependencies

These are required for the Ping-Pong review loop to function. The Generator and Reviewer cannot operate without them.

| Dependency | Role | Installation |
|-----------|------|-------------|
| **karpathy-guidelines** | Core review framework used by the Reviewer subagent. Provided by the `andrej-karpathy-skills` plugin. | `/plugin marketplace add forrestchang/andrej-karpathy-skills`<br>`/plugin install andrej-karpathy-skills@karpathy-skills` |
| **superpowers** | Plugin providing subagent workflow skills (brainstorming, systematic-debugging, TDD, etc.) that the Generator auto-routes to during plan construction. | `/plugin install superpowers@claude-plugins-official` |

### Optional Dependencies

These enhance post-execution workflows but are not required for the core review loop. The skill degrades gracefully without them (Phase 3 is skipped).

| Dependency | Role | Installation |
|-----------|------|-------------|
| **closure-knowledge-distillation** | Called in Phase 3 for post-execution knowledge distillation. A locally-developed companion skill that turns completed review decisions into structured knowledge assets. | No public repository. This is a companion skill maintained within the same skill ecosystem. If not installed, Phase 3 is simply skipped -- the core review and execution loop is unaffected. |

Without the core dependencies, the Generator cannot auto-route to the right workflow and the Reviewer has no review framework. Without the optional dependency, knowledge closure (Phase 3) is skipped but the Ping-Pong review and atomic execution (Phases 0-2) continue normally.

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

After installation, verify the core dependencies listed above are also installed. The skill will report which dependencies are missing on first activation.

## Triggers

The skill activates automatically when:

- Plan mode is entered (`EnterPlanMode`)
- A user writes a plan file
- A user says "review this plan" or "Karpathy review"
- A multi-step complex task is detected -- the skill self-determines activation when it sees: multiple target files involved, cross-file references or ID changes, or a plan that would benefit from adversarial review before execution

It also serves as the quality gate in the `writing-plans` → `karpathy-plan-review` → `executing-plans` superpowers pipeline.

## Relationship with karpathy-guidelines

These two skills are complementary, not alternatives. `karpathy-guidelines` is the reviewer subagent's core framework within this skill. It is provided by the `andrej-karpathy-skills` plugin (`forrestchang/andrej-karpathy-skills`).

| | karpathy-guidelines | karpathy-plan-review |
|---|---|---|
| **Scope** | Code (single line / function / file) | Plans / designs (multi-step / multi-file) |
| **Timing** | While coding | After planning, before execution |
| **Mechanism** | 4-principle behavioral guidance | Multi-agent Ping-Pong + convergence + grep verify |
| **Verification** | Test-driven (write test → make pass) | grep-hardened (grep / awk / wc checkable) |
| **Agent model** | Single agent, self-discipline | Two agents, adversarial review |
| **Do-not-change** | Avoids touching unrelated code | Explicit do-not-change list with reasons |

**Use both.** `karpathy-guidelines` keeps your code clean day-to-day. `karpathy-plan-review` ensures your plans survive adversarial scrutiny before you invest tokens in execution.

## Anti-Patterns

A non-exhaustive list of thoughts this skill is designed to reject:

| Thought | Reality |
|---------|---------|
| "This plan is simple enough, skip review." | All plans are reviewed. Simple ones may pass in one round. None are skipped. |
| "One more round just to be safe." | Convergence is convergence. Review has a cost. Respect the criteria. |
| "I will fix it during execution." | If it is not in the plan, it is not in the execution. Untracked changes are untraceable bugs. |
| "That assumption is too obvious to write down." | All assumptions are explicit. "None" is not a valid answer. |
| "One alternative is enough." | Fewer than 3 alternatives = insufficient exploration. |
| "Two agents cost too many tokens." | In practice, the cost of independent review is lower than the cost of debugging a broken plan in production. |
| "I will just adopt the reviewer's suggestion directly." | The reviewer finds problems. The generator produces solutions. Mixing the two roles recreates the self-review problem. |
| "Ping-Pong is too slow, skip to execution." | An unconverged plan is not executable. Skipping review = skipping the quality gate. |

## License

MIT — see [LICENSE](./LICENSE).

---

<div align="center">

**If this skill saves you from shipping a broken plan, consider giving it a star.**

Built on the insight that Karpathy's guidelines are powerful -- and even more powerful when enforced by an agent that is not you. See the `andrej-karpathy-skills` plugin (`forrestchang/andrej-karpathy-skills`) for the day-to-day code-level application of the same principles.

</div>
