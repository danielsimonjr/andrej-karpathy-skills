---
name: engineering
description: Behavioral guidelines to reduce common LLM coding mistakes, plus a Carmack addendum on measuring before optimizing and a systematic refactoring process. Use when writing, reviewing, or refactoring code - to avoid overcomplication, make surgical changes, surface assumptions, define verifiable success criteria, profile before optimizing, or refactor, clean up long functions, reduce nesting, remove duplication, break apart god classes, and modernize legacy code.
license: MIT
---

# Engineering Guidelines

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

## Refactoring

Systematic guidance for eliminating code smells and improving maintainability. Fires when the user wants to refactor code, clean up long functions, reduce nesting, remove duplication, break apart god classes, fix N+1 queries, address primitive obsession, or modernize legacy code for better structure and readability — while staying inside the four guidelines and the Carmack addendum above (surgical changes, measure before optimizing).

### Quick reference: code smells

| Smell | Symptom | Fix |
|---|---|---|
| God object/function | One thing doing everything | Split by single responsibility |
| Long parameter list | 5+ parameters | Extract parameter object |
| Deep nesting | 4+ levels of indentation | Guard clauses, extract method |
| Primitive obsession | Raw strings/ints for domain concepts | Create domain types |
| Feature envy | Method uses another class's data more | Move method to that class |
| Shotgun surgery | One change requires edits in many files | Consolidate related logic |
| Duplicate code | Copy-pasted blocks | Extract shared function |
| Magic numbers | Unexplained literal values | Extract named constants |
| Complex conditionals | Long boolean expressions | Extract predicate methods |
| N+1 queries | DB call per loop iteration | Batch query |

### Process: safest-first, three phases

**Phase 1 — Understand before touching.** Read and comprehend what the code actually does (not what it claims to do). Map data flows, dependencies, entry/exit points. Run existing tests (if none exist, write characterization tests first). Set specific, measurable improvement goals.

**Phase 2 — Apply refactorings, safest first.**
1. *Mechanical* (compiler-verified): rename for clarity, extract constants, extract/inline methods.
2. *Structural*: simplify conditionals (guard clauses), replace type codes with types, decompose complex functions.
3. *Data structure*: choose the right container, improve data locality (SoA vs AoS), normalize/denormalize.
4. *Performance* (measure first): eliminate hot-path allocations, hoist loop invariants, batch operations.

**Phase 3 — Verify after every change**, not just at the end: run all tests, verify behavior with real use cases and edge cases, measure performance if that was a goal, and review the diff — is it obvious? Would you understand this in six months?

### When NOT to refactor

- Code works, is stable, and rarely changes.
- You don't understand it yet — study first.
- No tests exist — write tests first.
- Under extreme time pressure.
- Code will be deleted soon.

### Checklist

- [ ] All tests pass
- [ ] Code is simpler than before (or measurably faster if complexity increased)
- [ ] No functionality changed (or changes are intentional and tested)
- [ ] Names clearly express intent
- [ ] Functions are small and focused
- [ ] Performance hasn't regressed
- [ ] The diff is reviewable

### Common mistakes

Refactoring without tests (write characterization tests first to lock behavior); changing too much at once (one small refactoring at a time, verify after each); optimizing without profiling (measure actual bottlenecks first — see the Carmack addendum); adding abstraction for one use case (wait for 2-3 similar implementations); refactoring code you don't understand (study it thoroughly first); breaking API contracts (identify and preserve all public interfaces).

See `examples/refactoring-quick-reference.md` for an 11-pattern before/after catalog (one best-fit language per pattern, covering code smells, performance, and architecture).
