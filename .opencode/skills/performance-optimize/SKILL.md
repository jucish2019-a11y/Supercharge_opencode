---
name: performance-optimize
description: Profile, identify bottlenecks, and optimize hot paths in application code
---

## What I do

I systematically identify and fix performance issues:

- Profile code to find actual bottlenecks (not assumed ones)
- Optimize algorithms, data structures, and hot loops
- Reduce unnecessary allocations, re-renders, and I/O
- Improve database query performance
- Resolve memory leaks and excessive resource usage

## When to use me

Use this skill when:
- The application is slow but the cause is unknown
- A specific operation takes too long
- Memory usage is growing unbounded
- The user reports laggy UI or slow API responses
- Loading time exceeds acceptable thresholds

## How I work

1. **Measure first** — Never assume where the bottleneck is. Use profiling tools, benchmarks, or timing logs to identify the actual slow path. Common tools: browser DevTools, `console.time`, Python `cProfile`, Node `--prof`, `perf`, `ab`, `wrk`.
2. **Reproduce reliably** — Create a consistent benchmark or test case that demonstrates the performance issue. Record baseline metrics.
3. **Analyze the bottleneck** — Categorize the issue:
   - **CPU-bound**: Inefficient algorithm, excessive computation, tight loops
   - **I/O-bound**: Unnecessary network/disk calls, N+1 queries, missing batching
   - **Memory-bound**: Leaks, excessive allocation, retained references, GC pressure
   - **Render-bound**: Unnecessary re-renders, layout thrashing, large DOM updates
4. **Plan optimizations** — Prioritize by impact. Consider:
   - Algorithmic improvement (lower time/space complexity)
   - Caching and memoization
   - Batching and parallelization
   - Lazy loading and code splitting
   - Index addition or query restructuring
   - Data structure change (array → map, list → set)
5. **Implement one change at a time** — Apply the most impactful optimization first. Remeasure after each change.
6. **Verify** — Confirm the optimization improves the benchmark without breaking functionality. Run the full test suite.

## Anti-patterns to avoid

- Premature optimization — only optimize measured bottlenecks
- Micro-optimizations that hurt readability for negligible gains
- Adding caches without invalidation strategies
- Optimizing without a baseline measurement

## Output format

After optimization:

```
## Performance Optimization Report

### Baseline
- Metric: X ms / X MB / X req/s

### Changes
1. [Description] → saved X ms (Y% improvement)
2. [Description] → saved X ms (Y% improvement)

### Final
- Metric: X ms / X MB / X req/s (Z% improvement overall)
```