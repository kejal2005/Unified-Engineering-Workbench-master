# PROMPT 10: Parallel batch engine (Stage 6)

## Requirements
1. `BatchSpec{caseCount, meshElements, seed}` with validation. Generator uses `std::mt19937_64` and explicit `uniform_real_distribution` draws made by a portable transformation. Varied positive k (e.g. 1-100) and q (e.g. 1e3-1e5); other parameters fixed. Generate full case list first, before timing.
2. `ThreadPool`: bounded, fixed workers, task queue, `submit`/`wait`, destructor joins, no detached threads. Parallel runner partitions case indices in chunks (or atomic next-index counter); results written to pre-sized vector by index (deterministic ordering, no result lock). Cancellation checked between cases.
3. `SerialRunner` and `ParallelRunner(p)` call the same `solveBatchCase`. Output: per-case temperature vector. `maxAbsDifference(serial, parallel)`.

## Tests
- Positive: same seed -> identical case list; serial vs parallel(1,2,3,4) max diff <= 1e-8 (expect 0 exactly); order preserved.
- Negative: caseCount=0, mesh=0, negative workers, workers > cap -> clamped or error per documented rule.
- Edge: caseCount < workers; caseCount=1; worker task throwing -> propagated as error, pool remains usable; cancel mid-run returns `cancelled`, no partial result published; stress test 200 repetitions.

## Acceptance
All pass; pool has no leaked threads (test thread count before/after).

## Stop If
Results depend on scheduling.

