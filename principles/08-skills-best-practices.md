# Skills Best Practices

Distilled from real-world skill development across 30+ custom skills.

## The Golden Rule

**The skill description is a trigger for the model, not a description for humans.**

```
BAD:  "Helps with servers."
GOOD: "Use when: ComfyUI hangs, GPU health check, SSH tunnel issues on port 2222."
```

The description determines WHEN Claude activates the skill. Include concrete trigger phrases that users actually say.

## Description Formula

```
[What it does] + [When to use -- specific trigger phrases] + [Key capabilities]
```

Example:
```yaml
description: |
  Parallel competency-based code review. Launches independent reviewers per competency
  (security, performance, architecture, database, concurrency, error-handling, frontend,
  testing). Use when: "deep review", "thorough review", "parallel review", "review by
  competency", or for large diffs (200+ lines).
```

## Required Sections

Every SKILL.md should include:

### Gotchas
Populate from real failures. Update every time an edge case is discovered.

```markdown
## Gotchas
- Agent tool limitation: subagents cannot launch sub-subagents
- Context size: if competency touches >20 files, prioritize critical ones
- False positives: parallel agents don't share context, synthesis step catches these
```

### Troubleshooting
Symptom -> Cause -> Solution format.

```markdown
## Troubleshooting
- **Agent times out** -> Scope too broad -> Reduce file count per competency
- **Duplicate findings** -> Multiple competencies overlap -> Synthesis step deduplicates
```

## File Structure

```
skill-name/
  SKILL.md          # Main file (keep under 5000 words)
  references/       # Detailed reference material
  scripts/          # Validation and helper scripts
```

- No `README.md` inside skill folders (SKILL.md IS the readme)
- Heavy reference material goes in `references/`, not inline
- Scripts go in `scripts/`, not embedded in SKILL.md

## Critical Validations: Scripts, Not Words

Code is deterministic. Language is not.

```
BAD:  "Make sure all tests pass before proceeding"
GOOD: A shell step that runs pytest and checks exit code
```

If a validation can be expressed as a script, it should be a script. Reserve natural language instructions for creative work (writing code, analysis, design decisions).

## YAML Frontmatter

```yaml
---
name: skill-name
version: 1.0.0
description: |
  [trigger-optimized description]
allowed-tools:
  - Bash
  - Read
  - Edit
  - Grep
  - Glob
  - Agent
---
```

- `allowed-tools`: list only what the skill actually needs
- `description`: multi-line, include trigger phrases
- `version`: bump on significant changes

## Anti-Patterns

1. **Monolithic skills**: a 10,000-word SKILL.md that covers everything. Break into sub-skills or use references/.
2. **Vague descriptions**: "Helps with development" -- too broad, won't trigger correctly.
3. **No Gotchas section**: means the skill hasn't been battle-tested.
4. **Embedded scripts**: long bash scripts inside SKILL.md. Move to scripts/.
5. **Human-oriented docs**: writing for a person to read, not for a model to execute.
