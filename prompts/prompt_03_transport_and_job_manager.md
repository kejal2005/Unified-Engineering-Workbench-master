# PROMPT 3: Transport layer, job manager and server lifecycle (Stage 3)

## Objective
Implement the loopback HTTP server, error mapping, job system and shutdown.

## Inspect
`core`, `ARCHITECTURE.md` API section.

## Requirements
1. `src/main.cpp`: parse `--port 0`, `--launch-ts`, `--web-root`. Bind `127.0.0.1` only. Print the chosen port in one stable line (`PORT=<n>`).
2. `transport`: route registry, JSON helpers, `ErrorMapper` (core `Error` -> status + envelope), body size limit from Config, exception boundary mapping unknown exceptions to `E_INTERNAL` (log details, no stack in the response).
3. Endpoints: `GET /api/health` (status, version, limits, `launchToReadyMs` computed from `--launch-ts` when provided, else null), `POST /api/heartbeat`.
4. `JobManager`: submit a callable receiving `(CancelToken&, ProgressSink&)`; states queued/running/completed/cancelled/failed; `GET /api/jobs/{id}`, `POST /api/jobs/{id}/cancel`. One mutex for the map; atomic progress; completed jobs retain results; bounded history (e.g. 32).
5. Static file serving from `--web-root` with path-traversal protection. Idle shutdown (configurable) after the heartbeat stops, with graceful job cancel and thread join.

## Files
`include/transport/*`, `src/transport/*`, `main.cpp`, tests.

## Constraints
No solver logic here. Core must still build without httplib.

## Tests
- Positive: health returns 200 and valid JSON. Job lifecycle with a test job (sleep and progress). Cancel mid-job -> `cancelled`.
- Negative: unknown job id -> 404 envelope. Oversized body -> 413. Path traversal (`/../`) -> 400/404.
- Edge: 100 rapid job submissions; cancel after completion (no error, no state change); shutdown during a running job joins cleanly (no hang, run under a 10 s timeout).

## Commands
Build and ctest; an integration script `scripts/smoke_server.bat` that starts the exe, curls health, kills it.

## Acceptance
All tests pass; the server is not reachable on a non-loopback interface; the idle shutdown test passes.

## Stop If
A data race or hang appears (run the job tests 50x in a loop).

