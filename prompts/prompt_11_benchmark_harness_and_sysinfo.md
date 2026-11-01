# PROMPT 11: Benchmark harness, system info and performance API (Stage 6)

## Requirements
1. `SystemInfo`: CPU brand string (`__cpuid` leaves), physical cores and logical processors (`GetLogicalProcessorInformationEx`), total RAM (`GlobalMemoryStatusEx`), OS version, plus compiler and optimisation flags from `build_info.hpp`. Missing values reported as "unavailable", never guessed.
2. `Benchmark`: workers list = `{1,2,3,4}` capped to logical CPUs. For each: 1 warm-up run (not counted), 5 measured repetitions with `steady_clock` around the compute only. Median, min, max. `speedup = T1/Tp`, `efficiency = speedup/p`. Verify max abs difference vs serial for every p, and fail job if > 1e-8. If logical CPUs == 1: run parallel path with 2 workers for demonstration and set `speedupAssessable=false` with disclosure string.
3. Job API: `POST /api/jobs/performance` -> progress per repetition and cancel. Result contains system info, workload, per-p times, stats, differences, `launchToReadyMs` (separate field).
4. `POST /api/export/benchmark` -> CSV (raw repetition times) and JSON; default-run outputs saved to `results/benchmarks/`. Default workload sized so serial median >= 0.5 s.

## Tests
- Positive: median/speedup/efficiency on injected synthetic timings.
- Negative: maxWorkers=0 or 17, case count huge (limit error).
- Edge: logical CPUs mocked as 1, 2, 3 via injectable provider; cancel mid-benchmark; two benchmark jobs started concurrently (second rejected or queued).

## Acceptance
Real run produces files in `results/benchmarks/`; record real numbers only.

## Stop If
Timings include generation or serialization.

