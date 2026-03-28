# Code Review Strategies

## Problem

Code review by LLMs ranges from superficial ("looks good") to expensive deep analysis. The challenge is matching review depth to the stakes: daily PRs need speed, pre-release changes need thoroughness, and security-sensitive code needs specialized scrutiny. A single approach cannot optimize for all three.

## Quick Comparison

| Aspect | A: Sequential Checklist | B: Parallel Competency | C: Cross-Model Adversarial | D: LLM + Static Analysis |
|--------|------------------------|----------------------|--------------------------|-------------------------|
| **Agents** | 1 | 3-6 parallel | 2 (different models) | 1 LLM + 1 tool |
| **Review depth** | Broad, shallow | Deep per domain | Broad, diverse | Mechanical + judgment |
| **Speed** | Fast (2-5 min) | Slow (10-20 min) | Medium (5-10 min) | Fast (3-5 min) |
| **Cost per review** | ~$0.50-2 | ~$5-15 | ~$2-5 | ~$1-3 + tool license |
| **Catches cross-cutting issues** | Rarely | Yes (synthesis step) | Sometimes | Only if rules exist |
| **Model blind spots** | Present | Present (same model) | Mitigated (different models) | Mitigated (tool has no blind spots for its rules) |
| **Setup** | Minimal | Skill definition | Codex CLI + Claude | Semgrep/ESLint config |
| **Best for** | Daily PRs <200 lines | Large changes, pre-release | High-stakes, security | All repos as baseline |

---

## A: Sequential Checklist Review

**Source:** gstack /review pattern, standard single-agent review

**Core idea:** One agent, one checklist, sequential passes. The reviewer walks through the diff checking categories in order: correctness, security, performance, style, tests. Fast and predictable.

**Typical checklist:**
1. Correctness -- does the code do what the PR claims?
2. Security -- injection, auth bypass, data exposure
3. Performance -- N+1 queries, unnecessary computation, memory leaks
4. Error handling -- edge cases, failure modes, error messages
5. Style -- naming, structure, consistency with codebase
6. Tests -- coverage, edge cases, test quality

**Pros:**
- [+] Fast -- completes in 2-5 minutes for typical PRs
- [+] Predictable output format -- same categories every time
- [+] Low cost -- single agent session
- [+] Easy to customize checklist per project
- [+] Good at catching known patterns (the checklist encodes team knowledge)

**Cons:**
- [-] Shallow per category -- 1 agent covering 6 areas cannot go deep on any
- [-] Single perspective -- one model, one context, one set of blind spots
- [-] Misses cross-cutting concerns (e.g., a performance issue that is also a security issue)
- [-] Checklist fatigue -- model may rush through later categories as context fills
- [-] Cannot specialize -- a security expert and a performance expert bring different knowledge

**When to choose:** Daily PRs under ~200 lines of diff. The team has a well-defined style guide. Changes are incremental (not architectural). You need fast turnaround -- review should not block CI.

---

## B: Parallel Competency Review

**Source:** /deep-review skill pattern (parallel specialist agents)

**Core idea:** First, scope the change to understand what is being modified. Then launch parallel specialist agents -- security reviewer, performance reviewer, architecture reviewer, test coverage reviewer, etc. Each gets focused context (only the files relevant to their domain). A synthesis step merges findings, deduplicates, and ranks by severity.

**Typical flow:**
1. Scope analysis -- classify the change (frontend, backend, infra, cross-cutting)
2. Launch parallel reviewers (3-6 depending on scope)
3. Each reviewer produces structured findings (severity, location, recommendation)
4. Synthesis agent merges, deduplicates, ranks, produces final report

**Pros:**
- [+] Deep per domain -- security reviewer only thinks about security, with full context
- [+] Catches cross-cutting issues in the synthesis step
- [+] Parallel execution means wall-clock time is not 5x slower
- [+] Each specialist can have domain-specific few-shot examples
- [+] Scales naturally -- add a reviewer for a new concern (accessibility, i18n, etc.)

**Cons:**
- [-] 5x cost compared to sequential review
- [-] Slower total time (10-20 minutes even with parallelism)
- [-] All agents share the same model's blind spots (Claude reviewing Claude's suggestions)
- [-] Synthesis step can lose nuance -- important detail buried in one specialist's output may be downranked
- [-] Overkill for small, obvious changes

**When to choose:** Large changes (500+ lines, or touching >5 files). Pre-release review where thoroughness justifies cost. Changes touching security-sensitive areas (auth, payments, data access). Architectural changes that affect multiple system components.

---

## C: Cross-Model Adversarial Review

**Source:** Multi-model review pattern (e.g., Codex + Claude)

**Core idea:** Run the same review on two different LLMs. Compare findings. Each model has different training data, different blind spots, different strengths. Disagreements between models are often the most interesting findings -- they highlight areas where one model catches what the other misses.

**Typical flow:**
1. Extract the diff and relevant context files
2. Send identical review prompt to Model A (e.g., Claude) and Model B (e.g., Codex/GPT)
3. Collect both sets of findings
4. Compare: what did A find that B missed? What did B find that A missed?
5. Focus human attention on disagreements

**Pros:**
- [+] Catches model-specific blind spots -- each model fails differently
- [+] Disagreements are high-signal -- they point to genuinely ambiguous or subtle issues
- [+] Simple to implement -- same prompt, two API calls
- [+] Builds confidence when both models agree (convergent validity)

**Cons:**
- [-] Requires access to two LLM providers (Codex CLI or OpenAI API + Claude)
- [-] Double the cost of single-model review
- [-] Comparison step adds complexity -- need to align different output formats
- [-] Both models may share the same blind spot (e.g., both miss a race condition)
- [-] Slower than single-model review

**When to choose:** High-stakes code (security, financial, safety-critical). You have access to multiple LLM providers. The cost of a missed bug in production far exceeds the review cost. You want to reduce dependence on any single model's judgment.

---

## D: LLM + Static Analysis

**Source:** Combined approach (Semgrep/ESLint/etc. + Claude review)

**Core idea:** Separate mechanical rules from judgment calls. Static analysis tools (Semgrep, ESLint, TypeScript strict mode, etc.) catch deterministic issues -- they never miss a pattern they are configured to detect. The LLM handles what tools cannot: architecture fit, naming quality, business logic correctness, test adequacy.

**Division of labor:**
- **Tool handles:** SQL injection patterns, XSS vectors, unused variables, type errors, import order, formatting, known vulnerability patterns
- **LLM handles:** "Is this the right abstraction?", "Does this test actually verify the behavior?", "Will this scale?", "Is the error message helpful?"

**Pros:**
- [+] Deterministic baseline -- static analysis never has a bad day, never skips a rule
- [+] LLM focuses on judgment calls where it adds the most value
- [+] Reduces false negatives -- tool catches what LLM might miss, LLM catches what tool cannot express
- [+] Static analysis is fast, cheap, and runs in CI without LLM costs
- [+] Tool rules encode team knowledge permanently (unlike LLM prompts which may drift)

**Cons:**
- [-] Requires Semgrep/ESLint setup and rule configuration (upfront investment)
- [-] Two separate systems to maintain (tool rules + LLM prompts)
- [-] Static analysis produces false positives that need tuning
- [-] Tool cannot adapt to novel patterns -- only catches what it is configured to catch
- [-] Integration between tool output and LLM review adds workflow complexity

**When to choose:** As a baseline for all repositories. The static analysis layer costs nearly nothing to run and catches a class of bugs deterministically. Layer LLM review on top for the judgment-call aspects. Especially valuable for teams with security compliance requirements.

---

## Recommendation

**Layer these approaches rather than choosing just one:**

1. **D (LLM + Static Analysis) as baseline for every repo.** Configure Semgrep or ESLint with security rules. This runs in CI, costs almost nothing, and catches deterministic bugs with zero false negatives for configured patterns.

2. **A (Sequential Checklist) for daily PRs.** Fast, cheap, good enough for incremental changes under ~200 lines. The checklist ensures nothing obvious is skipped.

3. **B (Parallel Competency) for significant changes.** Pre-release reviews, large refactors, security-sensitive areas. The 5x cost is justified by 5x depth.

4. **C (Cross-Model Adversarial) when available and stakes warrant it.** Not a daily tool, but valuable for code that handles money, auth, or personal data.

**Cost-optimized pipeline for a typical team:**
- Every PR: ESLint/Semgrep in CI (free) + Sequential checklist ($1)
- Weekly release candidate: Parallel competency review ($10)
- Security-critical changes: Cross-model adversarial ($5) + human reviewer
