---
name: performance
description: Use when writing or reviewing code with performance impact — database queries (N+1, SELECT *, indexes, pagination, execution plans), I/O in loops, async I/O, resource cleanup, caching, payload size, timeouts on external calls, front-end bundle and re-renders, or when asked to optimize something (rules R-PERF-01 to R-PERF-15).
---

# Performance

Performance is a design requirement, not a last-minute optimization. Any non-trivial optimization must be **measured** before and after.

## General

- **R-PERF-01** — Measure before optimizing (profiler, benchmark, metrics). No gut-feeling optimization on complex code.
- **R-PERF-02** — Right data structures (map/set for lookups, not a list scan); avoid avoidable O(n²).
- **R-PERF-03** — No I/O (network, disk, DB) inside a loop when batching is possible.
- **R-PERF-04** — Prefer asynchronous/non-blocking I/O.
- **R-PERF-05** — Release resources (connections, files, streams) deterministically (`using`, `try-with-resources`, `with`, `defer`, `finally`).

## Database

- **R-PERF-06** — No **N+1**: load required relations in a single query (join, eager load, batch `IN`).
- **R-PERF-07** — Select only needed columns; no `SELECT *` in production.
- **R-PERF-08** — Index columns used in frequent filters, joins and sorts.
- **R-PERF-09** — Paginate any potentially large list.
- **R-PERF-10** — Analyze the execution plan (`EXPLAIN` / `EXPLAIN ANALYZE`) of critical queries.

## Caching and network

- **R-PERF-11** — Cache expensive, rarely changing data, with an explicit invalidation strategy (TTL, event-based, versioned key) written down.
- **R-PERF-12** — Keep payloads small (compression, only useful fields).
- **R-PERF-13** — Set timeouts on all external calls.

## Front-end (if applicable)

- **R-PERF-14** — Limit unnecessary re-renders and bundle size (lazy loading, code splitting, memoization where measured).
- **R-PERF-15** — Optimize images and static assets (modern formats, sizing, compression).

## Optimization procedure

1. **Reproduce and measure** the baseline with a profiler, benchmark or production metrics. Record the numbers.
2. **Locate** the actual hotspot from the measurement — not from intuition.
3. **Fix** the cheapest structural cause first: N+1, I/O in loop, missing index, wrong data structure, missing pagination.
4. **Measure again** with the same setup. Keep the change only if the gain is real.
5. **Document** any optimization that hurts readability: why, and the measured gain (comment + PR description).
6. If the path is critical, add or update a benchmark/performance test so regressions alert (R-VER-06).

## Trade-off

Readability beats a micro-optimization with no measurable impact. Do not propose micro-optimizations without numbers.

## Review hunt list

Scan diffs for: queries or HTTP calls inside loops, lazy-loaded relations accessed in loops, `SELECT *`, unbounded `findAll`/list endpoints, new filter/sort columns without index, `list.contains`/`find` inside loops, sync I/O on request paths, unclosed connections/streams, HTTP clients without timeout, caches without invalidation, large response DTOs, heavy imports in front-end entry bundles.
