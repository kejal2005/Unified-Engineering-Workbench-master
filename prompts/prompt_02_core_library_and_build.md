# PROMPT 2: C++ core library and build system (Stage 2)

## Objective
Establish CMake, vendored dependencies and the `core` library with tests wired.

## Inspect
Root `CMakeLists.txt`, `third_party/`, `THIRD_PARTY.md`.

## Requirements
1. Vendor `nlohmann/json`, `cpp-httplib` and `GoogleTest` into `third_party/` with their license files. Record exact version and purpose in `THIRD_PARTY.md`.
2. `CMakePresets.json` with `debug` and `release` presets for MSVC x64. Release uses `/O2`, static CRT (`/MT`). Warnings as errors (`/W4 /WX`). Generate `build_info.hpp` (compiler, version, flags) from CMake for later benchmark reporting.
3. Implement `core`: `Error`/`ErrorCode` (codes from Part 3), `Result<T>` (value-or-error; no exceptions for expected failures), `Config` (limits: STL bytes, CSV bytes, max triangles, max nodes, max batch cases, idle timeout; defaults documented), `Logger` (thread-safe, to stderr and an optional file in a relative `logs/` dir), `CancelToken` (atomic flag).
4. A trivial `workbench_core` static library, `workbench_core_tests` executable, `enable_testing()`/CTest.

## Files
`CMakeLists.txt`, `CMakePresets.json`, `src/backend/CMakeLists.txt`, `include/core/*`, `src/core/*`, `tests/core/*`.

## Constraints
`core` must not include httplib. No global mutable state besides the logger's guarded sink.

## Tests
- Positive: `Result` holds value or error; Config defaults.
- Negative: accessing a value from an error `Result` fails cleanly.
- Edge: CancelToken concurrently set from 4 threads. Logger called from 8 threads without interleaving corruption.

## Commands
`cmake --preset release`, `cmake --build --preset release`, `ctest --preset release --output-on-failure`.

## Acceptance
Clean configure and build from scratch with no network; all tests pass; `dumpbin /dependents` on the tests exe shows only system DLLs.

## Stop If
The build needs a network, or the warnings-as-errors flag is disabled to compile.

