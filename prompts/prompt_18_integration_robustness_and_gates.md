# PROMPT 18: Integration, robustness and quality gates (Stage 10)

## Requirements
1. End-to-end API tests (C++ test executable running real server on ephemeral port): all endpoints, positive and negative.
2. Fuzz-style malformed inputs for both parsers (truncate at every byte of small valid file, random byte flips with fixed seed): must never crash or hang, must return structured error or valid parse.
3. Size-limit tests recorded as measurements (largest STL/CSV accepted, time, peak memory), written into `LIMITATIONS.md` and in-UI limits display.
4. Startup/shutdown tests: start/stop 50x; stop with running jobs; double-start of server on same port; orphan-process check.
5. Thread safety: repeat pool, job and hardware concurrency tests 200x; run AddressSanitizer/UBSan builds if toolchain supports them.
6. Full regression run; record all results in `TESTING.md` with test inventory mapped to requirement IDs.

## Acceptance
Every module has unit, negative and edge tests; zero flaky tests across 50 repeated full runs; traceability table in `TESTING.md` links each requirement to a passing test.

## Stop If
Any test is skipped or relaxed to pass.

