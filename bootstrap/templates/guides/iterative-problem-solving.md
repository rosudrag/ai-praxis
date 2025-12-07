# Iterative Problem Solving Guide

A structured approach to solving complex problems through hypothesis formation, testing, and refinement.

---

## The Core Loop

```
┌────────────────────────────────────────────────────────────────────────┐
│                                                                        │
│   ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐   │
│   │ HYPOTHESIZE│───▶│  ATTEMPT  │───▶│  EVALUATE │───▶│  REFINE   │   │
│   └───────────┘    └───────────┘    └───────────┘    └───────────┘   │
│         ▲                                                   │         │
│         └───────────────────────────────────────────────────┘         │
│                                                                        │
│   Continue until: Solution works OR hypotheses exhausted              │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

**Do not give up until you find a correct solution.** If one approach fails, understand why, then try another.

---

## Phase 1: Hypothesis Formation

Before writing any code, form explicit hypotheses about the problem.

### State the Problem Clearly

```
PROBLEM: [What behavior is expected vs what's happening]
CONTEXT: [Where in the codebase, what triggers it, when it occurs]
EVIDENCE: [Error messages, logs, test output, stack traces]
```

### Generate Multiple Hypotheses

Always generate **at least 2-3 hypotheses** before attempting a fix:

```
HYPOTHESIS 1 (simplest): [Description]
- Evidence for: [What supports this]
- Evidence against: [What contradicts this]
- How to test: [Concrete step to prove/disprove]

HYPOTHESIS 2: [Description]
- Evidence for: ...
- Evidence against: ...
- How to test: ...

HYPOTHESIS 3: [Description]
- Evidence for: ...
- Evidence against: ...
- How to test: ...
```

### Prioritization Rules

Test hypotheses in this order:

1. **Simplest explanation first** - Occam's Razor applies to debugging
2. **Most testable first** - If you can disprove it in 30 seconds, do that first
3. **Highest evidence first** - More supporting evidence = test sooner
4. **Least invasive first** - Prefer hypotheses that require smaller changes

---

## Phase 2: Structured Attempts

### Before Each Attempt

State explicitly:

```
TESTING HYPOTHESIS: [Which one]
APPROACH: [What I'm going to do]
EXPECTED OUTCOME: [What should happen if hypothesis is correct]
FAILURE INDICATOR: [What would prove hypothesis wrong]
```

### During Implementation

- Make the **minimum change** to test the hypothesis
- Don't fix multiple things at once
- Keep changes reversible until validated

### Capture Concrete Evidence

After each attempt, gather:

```
ACTUAL OUTCOME: [What actually happened]
MATCH EXPECTED: [Yes/No/Partial]
NEW INFORMATION: [What we learned]
```

---

## Phase 3: Evaluation

### The Diff Protocol

When comparing expected vs actual results, create explicit diffs:

```
EXPECTED:
[expected output/behavior]

ACTUAL:
[actual output/behavior]

DIFF:
[specific differences, line by line if applicable]
- Line 3: Expected "active", got "pending"
- Line 7: Expected 5 items, got 3 items
- Shape: Expected 10x10, got 10x8
```

### Soft Scoring

Even failed attempts have value. Score partial success:

| Score | Meaning |
|-------|---------|
| 0%    | Completely wrong (different shape, type error, crash) |
| 25%   | Right structure, wrong values |
| 50%   | Some correct outputs, some wrong |
| 75%   | Almost correct, minor differences |
| 90%   | Edge case failures only |
| 100%  | Fully correct |

**Why this matters:** A 75% correct solution is a better starting point for refinement than a 25% correct solution. Track progress across attempts.

### Decision Point

After evaluation:

```
IF hypothesis CONFIRMED:
  → Solution found, verify with additional tests, then done

IF hypothesis DISPROVED:
  → Update understanding, move to next hypothesis

IF hypothesis PARTIALLY CONFIRMED:
  → Refine hypothesis with new constraints, retry

IF all hypotheses exhausted:
  → Gather more information (logs, debugging, ask user)
  → Generate new hypotheses based on learnings
  → Continue loop
```

---

## Phase 4: Refinement

### Learning from Near-Misses

When an attempt gets close but not perfect:

1. **Identify the gap** - What specific cases fail?
2. **Categorize the failure** - Edge case? Logic error? Missing condition?
3. **Minimal fix** - What's the smallest change to close the gap?

### Structured Feedback Format

When refining, document what was learned:

```
ATTEMPT #[N] FEEDBACK:

What worked:
- [Specific things that were correct]

What failed:
- [Specific things that were wrong]
- [Root cause if known]

Score: [X]% correct

Refinement for next attempt:
- [Specific change to make]
- [Why this should fix the issue]
```

### Accumulating Solutions

Keep track of all attempts and their scores:

```
SOLUTION HISTORY:
1. [Approach A] - 50% correct - Failed on edge cases X, Y
2. [Approach B] - 75% correct - Failed on null input
3. [Approach C] - 90% correct - Failed on concurrent access
4. [Approach D] - 100% correct ✓
```

This history helps identify patterns and prevents repeating failed approaches.

---

## Common Problem Patterns

### Pattern: Test Passes Locally, Fails in CI

**Hypothesis candidates:**
1. Environment difference (env vars, paths, OS)
2. Race condition / timing sensitivity
3. Test isolation issue (shared state)
4. Missing dependency / different version

### Pattern: Works for Some Inputs, Not Others

**Hypothesis candidates:**
1. Edge case not handled (null, empty, boundary values)
2. Type coercion issue
3. Off-by-one error
4. Encoding / character set issue

### Pattern: Worked Yesterday, Broken Today

**Hypothesis candidates:**
1. Recent code change (check git log)
2. Dependency update
3. External service change
4. Data/state change

### Pattern: Error Message Doesn't Match Code

**Hypothesis candidates:**
1. Stale build / cache issue
2. Wrong file being executed
3. Error thrown from dependency, not your code
4. Async operation failing elsewhere

---

## Anti-Patterns to Avoid

### 1. Shotgun Debugging

❌ Making random changes hoping something works
✅ Systematic hypothesis testing with one change at a time

### 2. Confirmation Bias

❌ Only looking for evidence that supports your favorite hypothesis
✅ Actively seeking evidence that could disprove each hypothesis

### 3. Premature Optimization

❌ Fixing the "real problem" before confirming it's actually the problem
✅ Prove the hypothesis before implementing the fix

### 4. Abandoning Too Early

❌ Giving up after 1-2 failed attempts
✅ Exhausting hypotheses systematically, generating new ones from learnings

### 5. Not Capturing Learnings

❌ Fixing the bug and moving on
✅ Documenting what was learned for future similar problems

---

## Integration with TDD

This iterative approach **is** TDD applied to debugging:

| TDD Cycle | Problem Solving Equivalent |
|-----------|---------------------------|
| RED: Write failing test | HYPOTHESIS: State what should happen |
| GREEN: Make it pass | ATTEMPT: Implement minimal fix |
| REFACTOR: Clean up | REFINE: Improve solution quality |

The test IS the hypothesis. The implementation IS the experiment.

---

## Checklist: Before Asking for Help

Before escalating or asking for clarification:

- [ ] Stated the problem clearly with concrete evidence
- [ ] Generated at least 3 hypotheses
- [ ] Tested at least 2 hypotheses with concrete attempts
- [ ] Documented what was tried and what was learned
- [ ] Identified what specific information would help narrow down the cause

When you do ask for help, provide:
1. Problem statement
2. Hypotheses considered
3. Attempts made and their results
4. Current best guess
5. What specific help you need

---

## Quick Reference

```
1. HYPOTHESIZE
   - State 3+ possible causes
   - Rank by simplicity and testability
   - Identify how to test each

2. ATTEMPT
   - Test simplest hypothesis first
   - Make minimal, reversible changes
   - Capture concrete evidence

3. EVALUATE
   - Create explicit diffs (expected vs actual)
   - Score partial success (0-100%)
   - Decide: confirmed / disproved / partial

4. REFINE
   - Learn from near-misses
   - Document feedback
   - Generate new hypotheses if needed

5. REPEAT until solved or hypotheses exhausted
```

**Remember: Do not give up until you find a correct solution.**

---

## Related Guides

- [TDD Enforcement](tdd-enforcement.md) - Hypothesis-driven development via tests
- [Multi-Approach Validation](multi-approach-validation.md) - When to explore multiple solutions
- [Research Workflow](research-workflow.md) - Investigating unknowns systematically
