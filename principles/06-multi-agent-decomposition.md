# Multi-Agent Task Decomposition

Sources: [2603.14703] Multi-Agent System Optimization, [2603.13256] Training-Free Multi-Agent Coordination

## Core Principle

For tasks that span multiple files and cross-cutting concerns, a single agent becomes a bottleneck. Decompose into a coordinator + specialized sub-agents, each focused on one domain.

## Two Levels of Optimization

### Function-level
- One agent, one file
- Standard workflow, no decomposition needed
- Example: fix a bug in a single service

### System-level
- Multiple files, cross-cutting concerns
- Needs multi-agent coordination
- Example: add a feature that touches API, database, frontend, and tests

## Control-flow + Data-flow Representation

Before system-level optimization, build a dependency graph:
- Which components call which?
- What data flows where?
- This reveals bottlenecks and side effects invisible at function-level

## Lightweight Coordinator Pattern

Instead of one monolithic agent, use:

```
Coordinator (distributes tasks, does NOT write code)
    |
    +-- Frontend Agent (components, styling, routing)
    |
    +-- Backend Agent (API, business logic, validation)
    |
    +-- Infrastructure Agent (DB, migrations, config)
    |
    +-- Test Agent (unit tests, integration tests, e2e)
```

### Key Rules
- Coordinator distributes, does not implement
- Sub-agents are specialized by domain
- Coordination happens through **shared artifacts in the repo** (files, not conversation history)
- Each agent reads the state file, does its work, updates the state file

## Relationship to Harness Design

This extends Generator-Evaluator by adding a third role: the Coordinator. For tasks touching more than 3 files, consider multi-agent decomposition.

## When to Use

- Feature development spanning 3+ files
- Refactoring that touches multiple modules
- Any task where a single agent would need to context-switch between domains

## When NOT to Use

- Single-file changes
- Changes within one module/layer
- Tasks where the overhead of coordination exceeds the benefit
