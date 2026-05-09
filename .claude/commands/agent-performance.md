# /agent-performance

Investigate and improve performance with a coding agent: establish a baseline, profile to find the real bottleneck, make one change at a time, and measure the result before declaring improvement.

**When to use:** A specific operation is slow and you want to make it faster, or a performance regression has been reported and you need to find and fix it.

---

## Steps

### 1. Define the performance problem specifically

"The app is slow" is not a problem statement. Before starting:

- **What operation?** (page load, specific API endpoint, background job, search query)
- **How slow?** (current measured latency or throughput)
- **What's the target?** (acceptable threshold — p95 < 200ms, job completes in < 30s)
- **When did it become slow?** (always, since a recent change, under load)

Write this down:

```
Performance problem:
- Operation: GET /api/orders (order list page)
- Current: p95 = 2,400ms
- Target: p95 < 400ms
- Context: regression, was ~300ms before last week's deploy
```

Without a specific target, you don't know when you're done. Without a measured baseline, you can't tell if a change helped.

### 2. Measure before touching anything

Establish a reproducible baseline measurement before any change:

```bash
# HTTP endpoint — use a load testing tool
ab -n 100 -c 10 http://localhost:3000/api/orders
# or: k6, wrk, hyperfine for CLI tools

# Database query — use EXPLAIN ANALYZE
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC;

# Function — use the language's profiling tools
node --prof app.js  # Node.js
python -m cProfile -s cumulative app.py  # Python
```

Record the numbers. Do not rely on subjective "it feels faster." All future measurements must be compared against this baseline.

Checkpoint: Do you have a written baseline measurement you can compare against?

### 3. Profile to find the actual bottleneck

Do not guess. Profile:

```
Profile [operation] and find where time is actually being spent.
List the top 5 slowest operations by cumulative time.
Do not suggest fixes yet — just show where the time goes.
```

Common bottlenecks and how to find them:

| Symptom | Tool | What to look for |
|---------|------|------------------|
| Slow API response | Request profiler, APM | Which function calls take the most time |
| Slow database query | `EXPLAIN ANALYZE` | Sequential scans, missing indexes, N+1 queries |
| High memory / GC pressure | Heap profiler | Large allocations, retained objects |
| Slow page load | Browser DevTools Network tab | Large bundles, render-blocking resources, waterfall |
| Slow background job | Profiler + logging | Which step in the job takes longest |

Do not skip profiling. Optimizing the wrong thing wastes time and adds complexity for no gain.

Checkpoint: Can you point to a specific function or query that accounts for the majority of the time?

### 4. Make one change at a time

Once the bottleneck is identified, implement one fix:

```
Implement only this change: [specific optimization — add index on orders.user_id].
Do not change anything else yet.
```

One change at a time is not just tidiness — it's how you know whether the change worked. Multiple simultaneous optimizations make it impossible to attribute improvement (or regression) to a specific change.

Common fixes and their risks:

| Fix | Risk to watch for |
|-----|-------------------|
| Add database index | Index bloat, slower writes |
| Add caching | Stale data, cache invalidation bugs |
| Batch N+1 queries | More complex code, edge cases with empty sets |
| Reduce bundle size | Tree-shaking side effects, missing polyfills |
| Memoize expensive computation | Memory growth, stale results |

### 5. Measure the result — same method as the baseline

After the change, measure using exactly the same method as the baseline:

```bash
ab -n 100 -c 10 http://localhost:3000/api/orders
```

Compare the numbers. Write them down next to the baseline.

If the improvement is smaller than expected: investigate why before adding more changes. If it's worse: revert and re-examine the profiling data.

Checkpoint: Does the measurement show meaningful improvement over the baseline?

### 6. Iterate — one change at a time

If the target isn't met after the first fix, profile again. The bottleneck may have shifted:

```
After that fix, profile again.
What's the new top bottleneck?
```

Repeat: profile → fix → measure → profile again. Stop when the target is met or you've exhausted the obvious bottlenecks.

Do not add optimizations beyond the target threshold. Over-optimization adds complexity without user-visible benefit.

### 7. Document what you measured and what changed

Before closing the ticket:

```markdown
## Performance changes

**Baseline:** p95 = 2,400ms (GET /api/orders, 2024-01-15)
**Target:** p95 < 400ms
**Result:** p95 = 180ms

**Changes made:**
1. Added index on `orders(user_id, created_at DESC)` — reduced query from 1,900ms to 80ms
2. Added 60-second cache on order list for authenticated users — reduced DB calls by ~90%

**Trade-offs:**
- Order list is now up to 60 seconds stale — acceptable per product decision
```

This record is valuable: if a future change causes a regression, you know what the numbers were and what changed them.

---

## Exit criteria

- [ ] Problem defined: specific operation, current measurement, target threshold
- [ ] Baseline measured using a reproducible method before any change
- [ ] Profiling done to find the actual bottleneck — not guessed
- [ ] One change made at a time
- [ ] Each change measured using the same method as the baseline
- [ ] Target threshold met (or documented decision that it won't be met this cycle)
- [ ] Changes documented: what changed, what was measured, what trade-offs were accepted

---

## Common shortcuts to avoid

**"I know it's the database — let me just add some indexes."** Maybe. Or maybe it's an N+1 in the application layer calling the database 50 times. Profile first; the bottleneck is often not where you expect.

**"It feels faster now."** Measure it. Human perception of performance is unreliable, especially for changes in the 100–500ms range. Numbers are the only reliable signal.

**"Let me make several changes at once to save time."** If performance improves, you don't know which change caused it. If it regresses, you don't know which change to revert. One change, one measurement, always.

**"We hit the target, let's keep optimizing."** Stop at the target. Over-optimization adds complexity, makes the code harder to change, and often introduces correctness bugs (incorrect caching, skipped validations). Define the target before starting and stop when you hit it.
