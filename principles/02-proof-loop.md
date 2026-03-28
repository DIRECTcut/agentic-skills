# Proof Loop Pattern

Source: OpenClaw-RL paper (arxiv 2603.10165) + DenisSergeevitch/repo-task-proof-loop

## Core Principle

Next-state signals as universal proof. Test results, tool outputs, user reactions -- all of these are verification evidence. An agent cannot simply "claim" something is done -- it needs durable artifacts.

## Execution Protocol

```
spec freeze -> build -> evidence -> fresh verify -> fix -> verify again
```

1. **Spec freeze**: AC1, AC2... -- concrete acceptance criteria, frozen before implementation begins
2. **Build**: implement the minimal safe changeset
3. **Evidence**: builder collects proof (test output, logs, screenshots) in read-only mode
4. **Fresh verify**: a NEW session checks the repo state, writes verdict.json
5. **Fix**: minimal fixes based on problems.md, regenerate evidence
6. **Loop**: repeat until verdict = PASS on all acceptance criteria

## Four Sub-Agent Roles

Each role has strict boundaries -- no role can do another's job:

| Role | Can do | Cannot do |
|---|---|---|
| **Spec-freezer** | Read repo, define ACs | Touch code |
| **Builder** | Write code, collect evidence (read-only after build) | Verify own work |
| **Verifier** | Fresh session, write verdict only | See the build process |
| **Fixer** | Minimal fixes per problems.md | Sign off on final result |

## Key Differences from Harness Design

- All artifacts live **in the repository** (`.agent/tasks/`), not in conversation history
- Verifier is a **fresh session** -- not just a separate prompt, but entirely separate context
- Spec is frozen **before** the build (Harness Design's Sprint Contract can change mid-sprint)

## When to Use

- Mission-critical features (auth, payments, data integrity)
- Any change where "it works on my machine" is not acceptable
- When you need an audit trail of verification
- Multi-step implementations that must be correct end-to-end

## When NOT to Use

- Simple bug fixes (overhead not justified)
- Exploratory coding or prototyping
- Changes where automated tests already provide sufficient verification
