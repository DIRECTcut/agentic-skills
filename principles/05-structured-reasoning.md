# Structured Reasoning Protocol

Source: [2603.01896] Agentic Code Reasoning (Mar 2026)

## Core Principle

For complex tasks (debugging, architecture, optimization), replace free-form chain-of-thought with semi-formal reasoning. This eliminates "planning hallucinations" -- when the model builds plausible but incorrect reasoning chains.

## The Four Steps

### 1. Premises
What we know for certain -- facts from code, logs, tests, documentation.
- "Function X returns null when input is empty (line 42)"
- "The error occurs only after the 3rd retry (log timestamp 14:02:33)"
- "The migration added column Y but no index"

### 2. Execution Trace
Step-by-step tracing of control flow and data flow through the relevant code path.
- "Request enters at router.ts:15 -> middleware.ts:8 -> handler.ts:42"
- "Variable `state` is `pending` at line 42, changes to `active` at line 58"
- "The DB query at line 72 does NOT use the new index"

### 3. Conclusions
What formally follows from premises + trace. Only things that are logically entailed.
- "The null return at line 42 causes the downstream TypeError at line 89"
- "The missing index causes a full table scan on every request"

### 4. Rejected Paths
Hypotheses that were checked and eliminated, with the reason for rejection.
- "Hypothesis: race condition in the queue worker -- REJECTED: only one worker instance, confirmed via docker ps"
- "Hypothesis: cache stale data -- REJECTED: cache TTL is 60s, issue persists after 5min"

## When to Apply

- **Debugging**: when the cause is not obvious from the error message
- **Architecture decisions**: when there are more than 2 viable options
- **Performance optimization**: when the bottleneck is not obvious
- **Security review**: when trust boundaries are complex

## When NOT to Apply

- Simple CRUD tasks
- Changes where the solution is obvious
- Tasks where the user has already identified the root cause

## Why It Works

Structured prompting forces the model to:
1. Separate facts from assumptions
2. Trace actual execution rather than imagining it
3. Document what was ruled out (prevents revisiting dead ends)
4. Draw conclusions only from evidence, not from pattern matching
