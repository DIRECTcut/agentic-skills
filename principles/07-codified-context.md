# Codified Context -- Context as Infrastructure

Source: [2602.20478] Codified Context: Infrastructure for AI Agents in a Complex Codebase

## Core Problem

AI coding agents have no project memory -- every session starts from zero. CLAUDE.md and memory files help, but are insufficient for complex codebases when treated as passive documentation.

## The Principle

Context is infrastructure, not documentation.

| File | What it really is |
|---|---|
| `CLAUDE.md` | Runtime config for the agent, not a wiki |
| `.claude/rules/` | Conditional context injection, not a reference |
| Memory files | Persistent state, not notes |
| `PLAN.md`, `TODO.md` | Structured handoff artifacts, not logs |

## JIT Context Loading

The most important practice: load only what is needed for the current step.

1. Do NOT load the entire project into context upfront
2. Load only what is needed for the current step
3. Store step results in files, not in the chat
4. After compaction, re-inject only critical state

## The Context Engineering Pipeline

```
rules -> state -> JIT retrieval -> pruning -> compaction policy -> re-inject -> isolation
```

### Stage breakdown

- **Rules**: CLAUDE.md and `.claude/rules/` provide baseline context
- **State**: memory files, plan.md, todo.md provide current state
- **JIT retrieval**: read only the files needed for the current task
- **Pruning**: discard information that is no longer relevant
- **Compaction policy**: decide what to preserve when context is compressed
- **Re-inject**: after compaction, restore critical state from files
- **Isolation**: sub-agents get only their relevant slice of context

## Practical Guidelines

1. **Results go to files, not chat**: after any significant step, write the result to a file (plan.md, state.json, findings.md). This survives compaction.

2. **State files over memory**: for within-session tracking, use state files (plan.md with checkboxes). For cross-session knowledge, use memory.

3. **Scoped context for sub-agents**: when launching Agent tool, explicitly state what context to include. Not "everything" and not "nothing" -- the relevant slice.

4. **CLAUDE.md is not a changelog**: keep it focused on rules and principles that affect agent behavior. Move project history to memory files.

## When to Apply

- Any session longer than 30 minutes
- Multi-step tasks with state to track
- Tasks that may survive context compaction
- Projects with complex architecture where loading everything is impractical
