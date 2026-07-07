---
name: karpathy-guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. Use when writing, reviewing, or refactoring code to avoid overcomplication, make surgical changes, surface assumptions, and define verifiable success criteria. Includes a Carmack addendum on profiling before optimizing and simplifying performance-critical code.
license: MIT
---

# Karpathy Guidelines

Behavioral guidelines to reduce common LLM coding mistakes, derived from [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Carmack addendum — measure, then simplify

Complementary to the four guidelines above, drawn from John Carmack's engineering philosophy. Both sets fire on the same coding/refactor/perf tasks: Karpathy's guidelines govern *how* you approach the change; this addendum governs *what good looks like* once performance or low-level correctness is in play.

**Understand the system before touching it.** Cache behavior, pipelines, and memory/data layout determine real-world performance far more than intuition. Question "best practices" that haven't been measured in your actual context — cargo cult conventions are common and often wrong.

**Simplify relentlessly.** Every line is a liability. Write concrete, direct code first; add abstraction only after duplication is proven harmful (2-3 real variants), not for imagined future flexibility. Prefer explicit state over deep hierarchies and indirection.

**Measure, don't guess.** Never optimize before profiling. Roughly 90% of runtime lives in 10% of the code — find that 10% with real data, optimize it hard, and keep the other 90% clean and readable rather than "optimized" on speculation.

**Workflow for perf-sensitive work:** make it work (correct algorithm) → make it right (clean it up) → profile (find real bottlenecks) → optimize the hot path → verify correctness held → measure again to confirm the win.

**Failure modes to catch in review:** optimizing without profiling first; over-abstracting "for flexibility"; ignoring cache/data-layout effects (prefer structure-of-arrays over array-of-structures when loops touch few fields); premature SIMD/vectorization before the algorithm and layout are settled; complex error handling inside hot loops instead of validating at boundaries.
