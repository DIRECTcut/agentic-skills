# 06 - Multi-Agent Task Decomposition

**Source:** [2603.14703] Multi-Agent System Optimization + [2603.13256] Training-Free Multi-Agent Coordination

## Overview

Not every task needs multiple agents. The key question is: does this task involve cross-cutting concerns across multiple files and components, or is it contained within a single function or file? The answer determines whether a single agent suffices or whether decomposition into specialized sub-agents is warranted.

---

## Optimization Levels

### Function-Level (Single Agent)

One agent, one file (or a small set of closely related files). This is the standard workflow and covers the majority of development tasks.

**Characteristics:**
- Changes are localized to one module
- No cross-cutting concerns (security, performance, accessibility all within scope of one file)
- The agent can hold the full context in its working memory
- Verification is straightforward (run the tests for this module)

**Examples:**
- Fix a bug in a utility function
- Add a new API endpoint
- Update a database query
- Refactor a single component

### System-Level (Multi-Agent)

Multiple files, cross-cutting concerns, dependencies between components. A single agent either loses track of the big picture or misses side effects in distant files.

**Characteristics:**
- Changes span 3+ files across different modules
- Side effects in one component affect others (shared state, API contracts, database schema)
- No single agent can hold all relevant context simultaneously
- Verification requires checking multiple integration points

**Examples:**
- Adding a new feature that touches frontend, backend, and database
- Security hardening across multiple services
- Performance optimization that requires changes to caching, queries, and API layer
- Migrating from one library to another across the codebase

---

## Control-Flow and Data-Flow Representation

Before decomposing a system-level task, build a dependency graph. This step is often skipped and its absence is the primary cause of multi-agent coordination failures.

### What to Map

**Control flow:** Which components call which?
```
loginHandler() --> validateCredentials() --> database.query()
                --> createSession() --> redis.set()
                --> generateToken() --> jwt.sign()
```

**Data flow:** What data moves where?
```
User input --> loginHandler (email, password)
           --> validateCredentials (returns user object)
           --> createSession (writes session to Redis, returns session ID)
           --> generateToken (reads user.id, returns JWT)
           --> Response (JWT + session cookie)
```

### Why This Matters

Without the dependency graph:
- Agent A changes the return type of `validateCredentials()` without knowing Agent B depends on the old type
- Agent C optimizes a database query without knowing it feeds a cache that Agent D is also modifying
- The coordinator assigns work to agents based on file boundaries, missing the functional dependencies

With the dependency graph:
- Work boundaries follow data dependencies, not file boundaries
- Side effects are visible before they cause failures
- Integration points are identified upfront and assigned to specific agents

---

## Lightweight Coordinator Pattern

For system-level tasks, replace a monolithic agent with a coordinator plus specialized sub-agents.

### Architecture

```
                    Coordinator
                   (plans, assigns, integrates)
                  /      |       \        \
           Frontend   Backend    Infra    Tests
           Agent      Agent      Agent    Agent
```

### Coordinator Role

The coordinator:
- **Plans** -- breaks the task into sub-tasks based on the dependency graph
- **Assigns** -- gives each sub-task to a specialized agent
- **Integrates** -- combines results, checks for conflicts
- **Does NOT write code** -- the coordinator is a manager, not an implementer

### Sub-Agent Specialization

Each sub-agent handles a domain:

| Agent | Domain | Typical Files | Expertise |
|---|---|---|---|
| Frontend | UI, components, styles | `.vue`, `.tsx`, `.css` | Component APIs, reactivity, accessibility |
| Backend | API, business logic, auth | `.ts` (server), `.py` | REST conventions, validation, security |
| Infra | Docker, CI/CD, configs | `Dockerfile`, `docker-compose.yml`, `.yml` | Networking, volumes, environment variables |
| Tests | Test suites, fixtures | `*.test.ts`, `*.spec.py` | Testing patterns, mocking, coverage |

### Coordination Through Shared Artifacts

Sub-agents do NOT coordinate through conversation history. They coordinate through files in the repository:

- `PLAN.md` -- the coordinator's task breakdown with assignments
- `CONTRACTS.md` -- API contracts and interfaces between components
- `STATE.json` -- current progress, completed items, blockers
- Individual agent output files in designated directories

This means:
- Any agent can be replaced or restarted without losing coordination state
- The coordinator can check progress by reading files, not by remembering conversations
- Conflicts between agents are visible in the artifacts (two agents modifying the same contract)

---

## Decomposition Guidelines

### When to stay single-agent

- Task touches 1-2 files
- No cross-module dependencies
- Straightforward, well-defined changes
- The full context fits in one session

### When to decompose

- Task touches 3+ files across modules
- Changes in one file affect behavior in another
- Multiple specializations needed (frontend + backend + infra)
- The task is too large for one session's context window

### How to decompose

1. **Map dependencies** -- build the control-flow and data-flow graph
2. **Identify boundaries** -- find natural seams where components interact through defined interfaces
3. **Assign by domain** -- group related files into sub-tasks aligned with agent expertise
4. **Define contracts** -- specify the interfaces between sub-tasks (API shapes, data formats, shared state)
5. **Plan integration order** -- determine which sub-tasks must complete before others can start

---

## Relationship to Harness Design

Multi-Agent Decomposition extends the Generator-Evaluator pattern from [Harness Design (01)](01-harness-design.md):

| Pattern | Roles | When to Use |
|---|---|---|
| Solo agent | One agent does everything | Function-level tasks |
| Generator-Evaluator | Generator + Evaluator | Quality-critical single-domain tasks |
| Coordinator pattern | Coordinator + N specialized sub-agents | System-level multi-domain tasks |

The coordinator IS the third role. For tasks spanning more than 3 files, consider multi-agent decomposition. For tasks within a single domain but requiring quality assurance, the Generator-Evaluator suffices.

---

## Common Pitfalls

### 1. Premature decomposition

Splitting a simple task across multiple agents adds coordination overhead without quality benefit. If one agent can do it reliably, use one agent.

### 2. File-based boundaries instead of functional boundaries

Assigning "all .ts files to Agent A and all .py files to Agent B" ignores functional dependencies. The authentication flow might span both TypeScript and Python files. Boundaries should follow the dependency graph, not the file system.

### 3. Missing contracts

Without explicit interface contracts, sub-agents make incompatible assumptions. Agent A returns `{user_id: number}` while Agent B expects `{userId: string}`. Define contracts before parallel work begins.

### 4. Coordinator doing implementation

If the coordinator starts writing code, it loses its planning perspective. The coordinator should focus on decomposition, assignment, and integration -- not implementation.

---

## Relationship to Other Principles

| Principle | Relationship |
|---|---|
| **Harness Design (01)** | Decomposition extends Generator-Evaluator with a coordinator role |
| **Proof Loop (02)** | Each sub-agent's output can be verified independently through the proof loop |
| **Deterministic Orchestration (04)** | The coordinator's task assignment and integration checking can be partially deterministic (scripts checking contract compatibility) |
| **Codified Context (07)** | Shared artifacts (PLAN.md, CONTRACTS.md, STATE.json) are codified context that enables coordination without shared conversation history |
