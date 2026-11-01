# UNIFIED ENGINEERING WORKBENCH: IMPLEMENTATION BLUEPRINT

Source of truth: the attached PASS SCIENCE WORKS assignment. Where your master instruction and the assignment differ, I flag it and resolve it below.

---

# PART 1: ASSIGNMENT UNDERSTANDING

## 1.1 Mandatory requirements

| ID | Task | Core requirement |
|---|---|---|
| A | Interface and one-click | One UI with Geometry, Thermal, Performance, Hardware views. File selection, parameter validation, progress, actionable errors, export, responsive UI, safe cancellation. C++ core separate from presentation and transport. `Launch.exe` opens the app on Windows 11 x64 with no terminal, admin rights, installs or first-run build. VS Code or Antigravity run config starts the same packaged app. CMake source-build instructions are separate. |
| B | CAD/ANSYS visualization | ASCII STL (triangles, rotate/pan/zoom/fit). CSV `node_id,x_m,y_m,z_m,temperature_C` (coloured 3D point cloud, legend with units, node inspection). C++ parsing and validation. Reject malformed or non-finite data, explain empty files, document tested size limits. One sample of each with provenance. Self-made CSV labelled **synthetic**. |
| C | Thermal FEA | 1D steady-state, two-node linear elements. L=1 m, A=1e-4 m², k=10, q=10000, T(0)=100, T(L)=20. N=10/20/40/80. Plot T vs x against T_exact. **Max error at element midpoints via FE interpolation**, convergence, CSV export, assumptions, units, BCs, one debugging insight. |
| D | Parallel | Serial and parallel batches of independent solves in C++ (pool, threads or OpenMP). Fixed-seed generator with varied positive k and q, configurable case count and mesh size. Results in deterministic input order. Workers 1, 2, up to 4, capped at logical CPUs. Warm-up plus 5 reps, median, speedup, efficiency. CPU, physical/logical cores, RAM, compiler, flags, workload. Max diff ≤ 1e-8 °C. Launch-to-ready reported separately. Single-CPU case disclosed. |
| E | Embedded | Block diagram, interface/pin-voltage table (power, ground, voltage domains, level shifting, connectors, host comms, JTAG ownership, pinout assumptions). Explanation that JTAG is a debug interface. C++ mocks (discovery, capabilities, connect/disconnect, one telemetry message). All three device types shown in the UI plus one recoverable failure. Simulated behaviour labelled. Safe integration plan and limitations. |
| F | Backprop | Handwritten (scan, ≤2 pages): forward pass, four gradients, one simultaneous update, numbers, gradient-direction explanation. C++ without autodiff. Central differences, ε=1e-6, error ≤ 1e-5. Loss plot over 100 further steps at lr 0.1, with explanation. |
| ZIP | Delivery | `CandidateID_Workbench.zip` with Launch.exe, `src/`, CMakeLists, dependency versions, editor config, `data/`+provenance, `tests/`, `results/` (raw CSV and labelled plots), `docs/`, README, report ≤6 pages, 3–5 min video or timestamped screenshots with logs, Git bundle plus readable log, AI-use log, ownership statement. |

**Optional/not required:** native ANSYS/CAD formats, mesh-to-result registration, binary STL, physical hardware, minimum speedup, cloud deployment, any frontend language restriction.

## 1.2 Shortlist gates (critical)

1. Working C++ backend.
2. Standalone launch succeeds on a clean machine. Packaging-only failures get one 48-hour repackage with no new features.
3. C++ backend ≥ 6/12.
4. Thermal FEA ≥ 7/14.
5. Delivery/report ≥ 3/6.
6. Total ≥ 50 (interview band), ≥ 80 (strong band).

## 1.3 Discrepancies between your master instruction and the assignment (resolved)

| # | Issue | Resolution |
|---|---|---|
| D1 | Assignment requires `src/` containing frontend and C++ backend. Your tree puts `frontend/` and `backend/` at the root. | Use `src/frontend`, `src/backend` and `src/launcher`. Recorded in DECISIONS.md. |
| D2 | Your tree has `editor-config/`, but VS Code and Antigravity read `.vscode/`. | Use `.vscode/launch.json` and `tasks.json` as the live config. `editor-config/README.md` documents prerequisites. |
| D3 | Assignment says "dependency lock/version information". Your tree has no vendored-dependency location. | Add `third_party/` plus `THIRD_PARTY.md`, and vendor all C++ dependencies so builds work offline. |
| D4 | Master says "Eigen if useful". | Not used. A tridiagonal Thomas solver is exact for this 1D problem, faster and dependency-free. Dense reference solve is done in tests with a tiny hand-written Gaussian elimination. |
| D5 | The handwritten derivation cannot be AI-produced. | You write and scan it (≤2 pages). I only provide a numeric check sheet (Prompt 15). |
| D6 | The AI log needs ≥1 genuinely incorrect AI suggestion, or a documented challenge test. | I plan deliberate challenge tests (Prompts 8 and 10). You must record real outcomes only. |

## 1.4 Technical risks and hidden dependencies

| Risk | Mitigation |
|---|---|
| Evaluator has no VC++ redistributable | Static CRT (`/MT`). Verify with `dumpbin /dependents`. |
| Server process lingers after the browser closes | Launcher owns a Windows Job Object (kill-on-close), heartbeat idle-shutdown, and a single-instance mutex. |
| Browser launch variance | Launcher tries `msedge --app=URL`, then falls back to the default browser via `ShellExecute`. |
| Port conflicts | Bind `127.0.0.1` on port 0 (ephemeral), read the actual port back, pass it to the launcher. |
| SmartScreen and antivirus warnings on unsigned EXE | Documented in README. Unavoidable without a signing certificate. |
| Plotly bundle size | Use a partial bundle (`plotly.js-basic-dist-min`) if it covers line and scatter. Bundle locally, with no CDN. |
| Fonts from Google Fonts | Forbidden offline. Use system font stack or locally bundled woff2. |
| Midpoint error misread as nodal error | Nodal FE values are exact for this problem. Only midpoint error shows O(h²) convergence. Expected midpoint max errors are `q·h²/(8k)`: 1.25, 0.3125, 0.078125, 0.01953125 °C. |
| Parallel timing too short to be meaningful | Default workload sized so serial time is ≥ ~0.5 s. Report what was measured, even if speedup is poor. |
| Fabricated evidence | Results come only from files the backend writes. Docs reference those files. |
| Large file upload in JSON | Enforced size limits (limits are tested and recorded, never assumed). |
| Cancellation corrupting results | Cooperative cancel flag. Cancelled jobs return no partial results as final. |

---

# PART 2: FINAL TECHNOLOGY STACK

| Layer | Choice | Justification |
|---|---|---|
| Backend language | C++17 (MSVC primary, `/O2`) | Mandatory. C++17 is enough: `std::thread`, `<filesystem>`, `std::from_chars`. |
| Build | CMake ≥ 3.20 with presets | Required for source-build. Presets give reproducible Debug/Release. |
| HTTP | **cpp-httplib** (MIT, header-only) | Loopback REST with minimal dependency, easy static file serving, no Boost/Asio. |
| JSON | **nlohmann/json** (MIT, header-only) | Standard, safe parsing, simple contracts. |
| Tests | **GoogleTest** (BSD-3) plus CTest | Named tests, fixtures, parameterised tests. |
| Concurrency | `std::thread` bounded worker pool, `std::atomic`, `std::stop_token`-style flag | No OpenMP: avoids the MSVC runtime DLL and keeps pool behaviour explicit and testable. |
| Linear solver | Own tridiagonal (Thomas) solver | Exact for this mesh topology and fastest. Dense reference solver in tests only. |
| Windows system info | Win32 (`GetLogicalProcessorInformationEx`, `GlobalMemoryStatusEx`, `__cpuid`) | Real CPU, core and RAM data. |
| Launcher | Small C++ Win32 executable → `Launch.exe` | Native, no runtime needed. Packaging code only, with no backend features. |
| Frontend | React 18 + TypeScript + Vite + Tailwind | Specified. Vite output is pure static files, so evaluators need no Node. |
| 3D | Three.js (`BufferGeometry`, `OrbitControls`, `Points`, `Raycaster`) | Rotate/pan/zoom/fit, point picking. |
| Charts | Plotly.js (basic bundle) | Labelled axes with units, PNG export for `results/plots/`. |
| Data transport | REST + JSON, job polling | Simpler and more robust than WebSockets. No SSE complexity. |

**Dependency discipline:** every library is vendored, pinned in `THIRD_PARTY.md` (name, exact version, licence, purpose), and added only with a written reason in DECISIONS.md. I deliberately do not state version numbers here, because they must be verified when you vendor them.

## Offline architecture

```
Launch.exe ──spawns (hidden, Job Object)──► app/workbench_server.exe
     │                                         • binds 127.0.0.1:<ephemeral>
     │                                         • serves app/web/ (built React bundle)
     │                                         • serves /api/*  (C++ core)
     └── waits for /api/health ──► opens msedge --app=http://127.0.0.1:<port>/
```

Nothing touches the network. All assets (JS, CSS, fonts, Plotly, Three.js) are local. The server rejects non-loopback connections. Closing the window stops heartbeats, and after the idle timeout the server exits cleanly. The Job Object kills it if the launcher dies.

## Windows packaging

A scripted release step (`packaging/windows/build_release.bat`, build tooling only) does the following:
1. CMake Release build of server and launcher.
2. `npm run build`, copying `dist/` into `app/web/`.
3. Assemble the ZIP layout (below).
4. Run `dumpbin /dependents` to confirm no non-system DLLs.
5. Verify the static CRT.
6. Offline smoke-test (network adapter disabled or firewall block).

**ZIP layout:**
```
CandidateID_Workbench/
  Launch.exe
  app/ (workbench_server.exe, web/)
  src/ (frontend, backend, launcher)
  third_party/  CMakeLists.txt  CMakePresets.json  THIRD_PARTY.md
  .vscode/ (launch.json, tasks.json)
  data/ tests/ results/ docs/ README.md
  git/ (workbench.bundle, commit_log.txt)
```

---

# PART 3: COMPLETE ARCHITECTURE

## 3.1 System layers

```
┌──────────────────────────── Browser/app window ────────────────────────────┐
│ React UI: Geometry | Thermal | Performance | Hardware | Backprop           │
│ services/api.ts (typed contracts only; NO numerical code)                  │
└─────────────────────────────────┬──────────────────────────────────────────┘
                                  │ HTTP/JSON on 127.0.0.1
┌─────────────────────────────────▼──────────────────────────────────────────┐
│ TRANSPORT (backend/src/transport): routes, DTO↔JSON, error mapping,       │
│ JobManager, static files, heartbeat                                        │
├────────────────────────────────────────────────────────────────────────────┤
│ CORE LIBRARY (pure C++, no HTTP/JSON dependency except dto layer)          │
│ core: Result/Error, Config, Logger, CancelToken, ThreadPool                │
│ geometry: StlParser, NodalCsvParser, validators                            │
│ thermal: FemModel, Assembler, TridiagonalSolver, ExactSolution, Convergence│
│ performance: BatchGenerator, SerialRunner, ParallelRunner, Benchmark, Sys  │
│ hardware: IDeviceAdapter, Mock{Mcu,Fpga,Jtag}, DeviceRegistry              │
│ backpropagation: Model, Gradients, FiniteDifference, Trainer               │
└────────────────────────────────────────────────────────────────────────────┘
```

**Boundary rule:** core libraries never include `httplib.h`. Transport depends on core, never the reverse. The core is tested with GoogleTest without starting a server.

## 3.2 API contract

Base: `/api`. All responses are JSON, UTF-8.

| Method + path | Purpose | Body → Response |
|---|---|---|
| `GET /health` | Readiness | → `{status:"ready", version, launchToReadyMs, limits:{stlBytes,csvBytes,...}}` |
| `POST /heartbeat` | UI liveness | → 204 |
| `POST /geometry/stl` | Parse ASCII STL (raw text body) | → `{triangleCount, positions:[...], normals:[...], bounds, units:"as-provided"}` |
| `POST /geometry/nodal-csv` | Parse CSV | → `{nodeCount, ids, positions, temperatures, tMin, tMax, unit:"°C"}` |
| `POST /thermal/solve` | Run N list | `{elements:[10,20,40,80]}` → per-mesh nodes, midpoints, FE vs exact, errors, convergence orders |
| `POST /jobs/performance` | Start benchmark | `{caseCount, meshElements, seed, maxWorkers}` → 202 `{jobId}` |
| `GET /jobs/{id}` | Poll | → `{state:"queued\|running\|completed\|cancelled\|failed", progress:0..1, message, result?, error?}` |
| `POST /jobs/{id}/cancel` | Cancel | → 202 |
| `GET /hardware/devices` | Discover | → list with `simulated:true` |
| `POST /hardware/{id}/connect` / `/disconnect` / `/telemetry` | Operations | → state or telemetry. Timeout failure is injectable and recoverable by retry. |
| `GET /hardware/design` | Pin/voltage table data | static data from C++ |
| `POST /backprop/run` | Gradients, update, 100 steps | → forward values, analytic and numeric gradients, errors, update, loss series |
| `POST /export/{kind}` | CSV generation | → `text/csv` |

**Error envelope** (all failures):
```json
{"error":{"code":"E_NONFINITE_VALUE","message":"Line 14: temperature_C is 'nan'.","hint":"Remove or correct non-finite values and re-import.","details":{"line":14,"column":"temperature_C"}}}
```
Codes: `E_EMPTY_FILE`, `E_MALFORMED_FILE`, `E_NONFINITE_VALUE`, `E_SCHEMA_MISMATCH`, `E_TOO_LARGE`, `E_INVALID_PARAMETER`, `E_CANCELLED`, `E_DEVICE_TIMEOUT`, `E_NOT_FOUND`, `E_INTERNAL`. HTTP mapping: 400 validation, 413 size, 404, 409 state, 500 internal. No stack traces in responses. Details are logged server-side.

## 3.3 Lifecycle

**Startup:** Launcher (mutex check) → spawn the server in a Job Object → server loads config, binds port 0, writes the port to stdout/pipe → launcher polls `/health` → opens the window → UI starts heartbeat.
**Shutdown:** window closes → heartbeat stops → server idle timeout → cancels running jobs, joins pool, exits 0. The launcher also exits.

## 3.4 Resource, thread and error rules

- RAII everywhere. No raw `new`/`delete`. Pool destructor joins all threads.
- No mutable globals. `JobManager` guards its map with one mutex. Progress uses `std::atomic<double>`/`<int>`. Cancellation uses `std::atomic<bool>`.
- Each solve owns its own buffers (no sharing between cases). Output slots are pre-sized and indexed by case, giving deterministic order with no locking on results.
- Core returns `Result<T>`/`Error` for expected failures. Exceptions are caught only at the transport boundary and mapped to `E_INTERNAL`.
- One shared `Config` (limits, timeouts, idle shutdown) feeds core and transport.
- File input arrives only as bytes in the HTTP body. The core never opens arbitrary paths.

---

# PART 4: INITIAL REPOSITORY STRUCTURE

```
UnifiedEngineeringWorkbench/
├── README.md
├── CMakeLists.txt                  # top-level; adds src/backend, src/launcher
├── CMakePresets.json
├── .gitignore  .gitattributes  .editorconfig
├── .vscode/                        # launch.json, tasks.json (live editor config)
├── editor-config/                  # prerequisites doc for the run configuration
├── third_party/                    # vendored: httplib, nlohmann_json, googletest
├── THIRD_PARTY.md                  # name, version, licence, purpose
├── src/
│   ├── frontend/
│   │   ├── package.json  package-lock.json  vite.config.ts  tsconfig.json
│   │   ├── public/                 # local fonts, favicon
│   │   └── src/{app,components,features/{geometry,thermal,performance,hardware,backprop},
│   │             services,hooks,types,utils,styles}
│   ├── backend/
│   │   ├── CMakeLists.txt
│   │   ├── include/{core,geometry,thermal,performance,hardware,backpropagation,transport}/
│   │   ├── src/{core,geometry,thermal,performance,hardware,backpropagation,transport}/
│   │   ├── src/main.cpp
│   │   └── tests/                  # GoogleTest, one folder per module
│   └── launcher/                   # Launch.exe source (Win32)
├── data/{samples,provenance}/
├── tests/{integration,numerical,fixtures}/
├── results/{csv,plots,benchmarks}/
├── docs/{PROJECT_SPEC,ARCHITECTURE,IMPLEMENTATION_PLAN,PROJECT_STATE,DECISIONS,
│         TESTING,BUILD_AND_PACKAGING,LIMITATIONS,AI_USE_LOG,REPORT}.md
│   └── diagrams/  handwritten/  evidence/
├── scripts/                        # test runners, verification (no backend features)
└── packaging/{windows,release}/
```

**Differences from your reference tree:** `src/` parent (D1), `.vscode/` (D2), `third_party/` and `THIRD_PARTY.md` (D3), `src/launcher`, `src/backend/{include,src}/transport`, `docs/handwritten` and `docs/evidence`, and `.editorconfig`.

**Created first (Prompt 1):** `docs/PROJECT_SPEC.md`, `ARCHITECTURE.md`, `IMPLEMENTATION_PLAN.md`, `PROJECT_STATE.md`, `DECISIONS.md`, `.gitignore`, README stub, `git init`.

---

# PART 5: REQUIREMENTS TRACEABILITY MATRIX

| Requirement | Module | Prompt | Test evidence | Submission evidence |
|---|---|---|---|---|
| Four views in one UI | frontend/app | 4, 7, 9, 12, 14 | UI smoke (P18) | Screenshots, video |
| File selection | frontend + transport | 7 | Upload tests | Video |
| Parameter validation | core+UI | 3, 8–16 | Negative tests per module | TESTING.md |
| Progress/status/cancel | JobManager | 3, 11 | Cancel test (P11, P18) | Logs |
| UI responsive during compute | Jobs+polling | 11–12 | Poll-during-run test | Video |
| Result export | transport/export | 9, 12, 16 | CSV roundtrip tests | `results/csv` |
| C++ core separate from transport | architecture | 2–3 | Link-boundary check (core builds without httplib) | ARCHITECTURE.md |
| Launch.exe, no terminal/admin/install | launcher | 19 | Clean-machine checklist | Evidence log |
| Editor run configuration | `.vscode` | 19 | Manual run | editor-config doc |
| CMake build instructions | CMake | 2, 19 | Fresh clone build | BUILD_AND_PACKAGING.md |
| ASCII STL parse/validate | geometry | 5 | Unit, malformed, empty | Tests log |
| STL render, rotate/pan/zoom/fit | geometry UI | 7 | Manual script + UI test | Screenshots |
| Nodal CSV parse/validate | geometry | 6 | Schema, NaN/Inf, malformed | Tests log |
| Point cloud, legend, units, inspection | geometry UI | 7 | Manual + unit on colour map | Screenshots |
| Reject malformed/non-finite, explain empty | geometry | 5, 6 | Negative tests | Error screenshots |
| Tested file-size limits | geometry/config | 5, 6, 18 | Size boundary tests | LIMITATIONS.md |
| Samples + provenance + synthetic label | data | 6 | Provenance check | `data/provenance` |
| 2-node element K, f | thermal | 8 | Hand-checked matrices | TESTING.md |
| Assembly + both Dirichlet BCs | thermal | 8 | Dense reference solve, BC exact | Tests |
| FE interpolation at midpoints | thermal | 8 | Midpoint error = qh²/8k | Results CSV |
| N=10/20/40/80, plots, convergence | thermal+UI | 8, 9 | Order ≈ 2 | Plots |
| CSV export, assumptions, debugging insight | thermal/docs | 9, 20 | Export test | Report |
| Batch generator (seeded, positive k, q) | performance | 10 | Same-seed reproducibility | Tests |
| Serial + parallel, pool | performance | 10 | Agreement ≤ 1e-8, order test | Tests |
| 1–4 workers capped; warm-up; 5 reps; median; speedup; efficiency | benchmark | 11 | Stats unit tests | `results/benchmarks` |
| CPU/cores/RAM/compiler/flags/workload | sysinfo | 11 | Field-present test | Benchmark JSON |
| Launch-to-ready separate | launcher+server | 11, 19 | Value present in `/health` | Results |
| Single-CPU disclosure | benchmark/UI | 11, 12 | Simulated cap test | LIMITATIONS |
| Block diagram, pin/voltage table, JTAG explanation | hardware docs | 14 | Review checklist | `docs/diagrams`, report |
| Mock discovery/capabilities/connect/telemetry/timeout | hardware | 13 | State machine tests | Tests |
| 3 devices in UI + recovery, simulated labelling | hardware UI | 14 | UI test | Screenshots |
| Safe integration plan, limitations | docs | 14, 20 | Review | Report |
| Backprop forward/gradients/update | backprop | 15 | Numeric checks | Tests |
| Central diff, ε=1e-6, ≤1e-5 | backprop | 15 | 4 gradient tests | Results CSV |
| Loss plot 100 steps + explanation | backprop UI | 16 | Monotonic-decrease test | Plot |
| Handwritten ≤2 pages | docs/handwritten | 16, 20 | Checklist (human) | Scan |
| Robustness/startup/shutdown/offline | all | 18, 19 | Integration + offline | Evidence log |
| README, report ≤6 pp, video, git bundle, AI log, ownership | docs | 20 | Checklist | ZIP |
| Effort record | docs | 1, 20 | n/a | Report |

---

# PART 6: SEQUENTIAL IMPLEMENTATION PROMPT PACK

## How to use

For every prompt, paste **[PREAMBLE] + the prompt body + [FOOTER]**. The 15 mandatory sections are covered as follows: items 1–10 and 12 are in each body, and items 11, 13–15 plus the stop conditions are partly in the footer and partly prompt-specific.

### [PREAMBLE], paste at the start of every prompt
```
You are a Principal Software Architect and Senior C++ Engineer working on the
"Unified Engineering Workbench" (PASS SCIENCE WORKS internship assignment).
BEFORE ANY CHANGE: read docs/PROJECT_SPEC.md, docs/ARCHITECTURE.md,
docs/PROJECT_STATE.md and docs/DECISIONS.md. Inspect the actual repository
(list files; never assume a file exists). If the docs disagree with each other
or with the repo, STOP and report the discrepancy first.
Rules: C++17 backend only; all numerics, parsing and concurrency in C++; no Python
or Node backend; frontend never reimplements numerics; fully offline; no new
dependency without a DECISIONS.md entry; no TODO/FIXME or fake buttons;
comments only for non-obvious numerical/engineering decisions; relative paths only;
do not overwrite working code without inspecting it; never invent test results;
never weaken a failing test. Modify only the files this prompt names unless
a fix requires otherwise, and say why.
```

### [FOOTER], paste at the end of every prompt
```
COMPLETION PROTOCOL
1. Run the build and tests listed and paste the real output summary.
2. On failure: find root cause, fix the smallest responsible component, rerun
   the relevant tests, then the full regression suite, and record the result.
3. Update docs/TESTING.md (commands, expected, ACTUAL results), docs/PROJECT_STATE.md
   (phase, completed prompts, files created/modified, working vs not implemented,
   tests run and results, known bugs, blockers, next prompt number and exact text),
   docs/DECISIONS.md and docs/LIMITATIONS.md if affected, docs/AI_USE_LOG.md
   (real prompts/outputs/edits only).
4. Commit with a meaningful message ("P<n>: <summary>").
5. Do not mark this prompt complete unless every acceptance criterion passes.
6. Output a compact HANDOFF SUMMARY (done / not done / next prompt / risks).
```

---

### PROMPT 1: Requirements baseline and repository foundation (Stage 0–1)
**Objective:** Create the authoritative documentation set, git repo and folder skeleton. No application code.
**Inspect:** the assignment PDF, the master instruction, whether the repo exists.
**Requirements:**
1. `git init`. Add `.gitignore` (build dirs, `node_modules`, `dist`, IDE caches, `*.obj`, release staging), `.gitattributes` (LF for source, binary for samples), `.editorconfig`.
2. Create the full tree from Part 4 with `.gitkeep` only where a folder must exist empty.
3. Write `docs/PROJECT_SPEC.md`: every assignment requirement with ID (A1…F5, ZIP1…), numeric parameters verbatim, rubric table, shortlist gates, acceptance criteria. Include the expected midpoint-error values `q·h²/(8k)` for N=10/20/40/80 (1.25, 0.3125, 0.078125, 0.01953125 °C) as a derived check, marked "derived, to be verified by code".
4. Write `ARCHITECTURE.md` (layers, API contract, error envelope, lifecycle, thread rules, packaging), `IMPLEMENTATION_PLAN.md` (these 20 prompts), `DECISIONS.md` (D1–D6 and stack choices with reasons), `PROJECT_STATE.md` (template, Phase = P1 done), `TESTING.md`, `BUILD_AND_PACKAGING.md`, `LIMITATIONS.md`, `AI_USE_LOG.md` (header and honesty rules), README stub, and an effort log table (`docs/EFFORT_LOG.md`) for actual hours.
**Files:** all the above. **Constraints:** no code, no invented results. The spec must quote numbers exactly as in the PDF.
**Tests:** Positive: every file present, links resolve. Negative: grep spec for TODO or empty sections. Edge: spec parameter values diff-checked against the PDF by hand.
**Commands:** `git status`, `git log --oneline`, a directory listing.
**Acceptance:** all 10 docs exist and are consistent; the traceability IDs in the spec match `IMPLEMENTATION_PLAN.md`; the first commit exists.
**Stop if:** the spec omits any mandatory item or any number differs from the PDF.

### PROMPT 2: C++ core library and build system (Stage 2)
**Objective:** Establish CMake, vendored dependencies and the `core` library with tests wired.
**Inspect:** root `CMakeLists.txt`, `third_party/`, `THIRD_PARTY.md`.
**Requirements:**
1. Vendor nlohmann/json, cpp-httplib and GoogleTest into `third_party/` with their license files. Record exact version and purpose in `THIRD_PARTY.md`. (Since I cannot verify the current versions, record what you actually vendored.)
2. `CMakePresets.json` with `debug` and `release` presets for MSVC x64. Release uses `/O2`, static CRT (`/MT`). Warnings as errors (`/W4 /WX`). Generate `build_info.hpp` (compiler, version, flags) from CMake for later benchmark reporting.
3. Implement `core`: `Error`/`ErrorCode` (codes from Part 3), `Result<T>` (value-or-error; no exceptions for expected failures), `Config` (limits: STL bytes, CSV bytes, max triangles, max nodes, max batch cases, idle timeout; defaults documented), `Logger` (thread-safe, to stderr and an optional file in a relative `logs/` dir), `CancelToken` (atomic flag).
4. A trivial `workbench_core` static library, `workbench_core_tests` executable, `enable_testing()`/CTest.
**Files:** `CMakeLists.txt`, `CMakePresets.json`, `src/backend/CMakeLists.txt`, `include/core/*`, `src/core/*`, `tests/core/*`.
**Constraints:** `core` must not include httplib. No global mutable state besides the logger's guarded sink.
**Tests:** Positive: `Result` holds value or error; Config defaults. Negative: accessing a value from an error `Result` fails cleanly. Edge: CancelToken concurrently set from 4 threads. Logger called from 8 threads without interleaving corruption.
**Commands:** `cmake --preset release`, `cmake --build --preset release`, `ctest --preset release --output-on-failure`.
**Acceptance:** clean configure and build from scratch with no network; all tests pass; `dumpbin /dependents` on the tests exe shows only system DLLs.
**Stop if:** the build needs a network, or the warnings-as-errors flag is disabled to compile.

### PROMPT 3: Transport layer, job manager and server lifecycle (Stage 3)
**Objective:** Implement the loopback HTTP server, error mapping, job system and shutdown.
**Inspect:** `core`, `ARCHITECTURE.md` API section.
**Requirements:**
1. `src/main.cpp`: parse `--port 0`, `--launch-ts`, `--web-root`. Bind `127.0.0.1` only. Print the chosen port in one stable line (`PORT=<n>`).
2. `transport`: route registry, JSON helpers, `ErrorMapper` (core `Error` → status + envelope), body size limit from Config, exception boundary mapping unknown exceptions to `E_INTERNAL` (log details, no stack in the response).
3. Endpoints: `GET /api/health` (status, version, limits, `launchToReadyMs` computed from `--launch-ts` when provided, else null), `POST /api/heartbeat`.
4. `JobManager`: submit a callable receiving `(CancelToken&, ProgressSink&)`; states queued/running/completed/cancelled/failed; `GET /api/jobs/{id}`, `POST /api/jobs/{id}/cancel`. One mutex for the map; atomic progress; completed jobs retain results; bounded history (e.g. 32).
5. Static file serving from `--web-root` with path-traversal protection. Idle shutdown (configurable) after the heartbeat stops, with graceful job cancel and thread join.
**Files:** `include|src/transport/*`, `main.cpp`, tests.
**Constraints:** no solver logic here. Core must still build without httplib.
**Tests:** Positive: health returns 200 and valid JSON. Job lifecycle with a test job (sleep and progress). Cancel mid-job → `cancelled`. Negative: unknown job id → 404 envelope. Oversized body → 413. Path traversal (`/../`) → 400/404. Edge: 100 rapid job submissions; cancel after completion (no error, no state change); shutdown during a running job joins cleanly (no hang, run under a 10 s timeout).
**Commands:** build and ctest; an integration script `scripts/smoke_server.bat` that starts the exe, curls health, kills it.
**Acceptance:** all tests pass; the server is not reachable on a non-loopback interface (verify and document); the idle shutdown test passes.
**Stop if:** a data race or hang appears (run the job tests 50× in a loop).

### PROMPT 4: Frontend shell and typed API client (Stage 3/9 foundation)
**Objective:** Application shell wired to the real backend, with no fake data.
**Inspect:** `ARCHITECTURE.md` API contract; `src/frontend/` state.
**Requirements:**
1. Vite + React + TS (strict) + Tailwind. Lock dependencies (`package-lock.json`). Zero CDN or remote font use. Use a system font stack or local woff2.
2. Layout: fixed header (app name, backend status badge, version), left sidebar with five entries (Geometry, Thermal Analysis, Performance, Hardware, Backpropagation), main workspace, bottom status bar. Restrained neutral palette, one accent, defined spacing tokens.
3. `services/api.ts`: typed request helper, envelope parsing into `ApiError` (code, message, hint), `useJob` hook (poll every 250 ms, cancel, unmount cleanup), heartbeat every 5 s.
4. Shared components: `FileInput`, `NumberField` (label, units, min/max, inline error), `StatusBanner` (idle/running/success/error), `ErrorPanel`, `DataTable`, `EmptyState`.
5. Each view currently shows an honest empty state ("No data loaded") and real backend connectivity. No fake controls.
6. Vite proxy to the backend for dev. Production build uses relative URLs.
**Constraints:** no numerical code in TS. Keyboard accessible, labelled inputs.
**Tests:** Positive: component tests for `NumberField` validation and `ApiError` parsing (Vitest, dev-only dependency). Negative: backend unreachable → clear status badge, not a crash. Edge: a `useJob` unmount while polling leaves no leaked timers.
**Commands:** `npm ci`, `npm run build`, `npm test`, and run the server against `dist/`.
**Acceptance:** the built bundle served by the C++ server shows the shell; the status reflects the real `/health`; the production build makes no external requests (verify in the network panel and document).
**Stop if:** any asset loads from the internet.

### PROMPT 5: ASCII STL parser (Stage 4)
**Objective:** Robust C++ STL parsing and validation plus the endpoint.
**Requirements:**
1. Parse ASCII STL (`solid … facet normal … outer loop … vertex … endloop endfacet … endsolid`). Case-insensitive keywords, arbitrary whitespace, scientific notation via `std::from_chars` (or an equivalent locale-independent method).
2. Validate: non-empty; at least one facet; each facet has exactly 3 vertices; all numbers finite; facet count ≤ Config limit; size ≤ limit. Binary STL detection (e.g. starts with `solid` but contains non-text bytes, or size = 84 + 50n pattern) returns an actionable error: "Binary STL is not supported; export ASCII STL." Do not silently misparse.
3. Output: flat position array (float32-compatible), computed face normals when the file normal is zero or non-unit, bounds, triangle count, degenerate-triangle count (reported, not rejected).
4. `POST /api/geometry/stl` (raw text body). Errors carry line numbers.
**Files:** `include|src/geometry/StlParser.*`, transport route, `tests/geometry/*`, `tests/fixtures/stl/*`.
**Tests:** Positive: single triangle; cube (12 facets); scientific notation; CRLF. Negative: empty file, whitespace-only, missing `endloop`, 2 vertices, text instead of number, `nan`/`inf`, binary STL, truncated file. Edge: multiple solids; zero-area triangle; very long line; a file at exactly the limit and limit+1; Unicode BOM.
**Acceptance:** all fixtures behave as specified; every error has code, line and hint; the largest passing synthetic STL (generated by a script) is recorded with parse time and memory in TESTING.md as a *measured* size limit.
**Stop if:** any malformed fixture is accepted.

### PROMPT 6: Nodal temperature CSV parser and sample data (Stage 4)
**Objective:** CSV parsing with an exact schema, plus the sample files with provenance.
**Requirements:**
1. Header must match `node_id,x_m,y_m,z_m,temperature_C` exactly (trim whitespace, tolerate a UTF-8 BOM and CRLF, reject any other header with an `E_SCHEMA_MISMATCH` message showing expected vs found).
2. Each row: 5 fields; `node_id` a non-negative integer (reject duplicates); coordinates and temperature finite doubles; reject empty cells. Line-numbered errors. Empty file → `E_EMPTY_FILE` with an explanation; header-only → "no data rows".
3. Output: arrays plus `tMin`, `tMax`, node count, units (`m`, `°C`).
4. `POST /api/geometry/nodal-csv`. Limits per Config.
5. Create `data/samples/sample_part.stl` (small, self-authored, a simple solid with a few dozen facets) and `data/samples/sample_nodal_temperature_SYNTHETIC.csv` (analytically generated field, e.g. a smooth gradient on a grid; the filename and header comment in provenance must say SYNTHETIC). Write `data/provenance/*.md` (author, creation method, units, schema notes, licence, "not an ANSYS export"). The generator, if any, lives in `scripts/` and is only a data-creation tool.
**Tests:** Positive: valid file, BOM, CRLF, columns with extra spaces, negative temperatures. Negative: wrong header, missing column, extra column, `NaN`, `Inf`, `1e999`, text in numeric field, duplicate id, negative id, empty/header-only. Edge: all temperatures equal (tMin==tMax, legend must still work, so the API returns the values and the UI handles it in P7); a single node; a size-limit boundary.
**Acceptance:** the samples load through the real endpoint; provenance exists for both; measured CSV limits are recorded.
**Stop if:** the synthetic CSV is not labelled synthetic in both file name and provenance.

### PROMPT 7: Geometry view, 3D viewer (Stage 4/9)
**Objective:** Fully interactive viewer driven by backend-parsed data.
**Requirements:**
1. File pickers for STL and CSV; show file name, size, parse status. Errors displayed with code, message and hint.
2. Three.js scene in a React component with proper disposal (geometries, materials, renderer, listeners). STL as a `BufferGeometry` mesh with simple shading plus optional edges; rotate (left), pan (right/shift), zoom (wheel), **Fit to view** button (bounding sphere), reset.
3. CSV as `Points`, coloured by temperature via a perceptually reasonable colour map defined in `utils` (pure function with unit tests). Legend: gradient bar, tMin/tMax, unit "°C", and handling for tMin==tMax. Point-size control.
4. Node inspection by raycasting (`Points` threshold scaled to bounds): show node_id, x/y/z in m, temperature in °C, with highlighted marker. Toggle between STL and cloud visibility. Axes/scale indicator in model units. Label the STL units as "as provided in file (not specified by STL format)".
5. A banner when the loaded CSV is the sample synthetic file (based on the header/file name, backed by a backend flag if you prefer; state your choice).
**Tests:** Positive: colour map endpoints and monotonicity, bounds fit calculation. Negative: bad files show errors without crashing the viewer. Edge: a single point, a degenerate (flat) bounding box, 100k points interaction (record the measured frame behaviour honestly).
**Acceptance:** manual script in TESTING.md executed and recorded with screenshots; no console errors; memory does not grow on repeated loads (check and record).
**Stop if:** fit-to-view fails for the sample or a flat box.

### PROMPT 8: Thermal FEM core (Stage 5)
**Objective:** Correct 1D FEM solver in C++ with verification, no UI.
**Requirements:**
1. `ThermalProblem{L,A,k,q,T0,TL}` with validation (all positive/finite; N ≥ 1). Defaults exactly: L=1, A=1e-4, k=10, q=10000, T0=100, TL=20.
2. Two-node element: `Ke = (kA/h)[[1,-1],[-1,1]]`, `fe = (qAh/2)[1,1]`. Global assembly into tridiagonal storage (assembly loop over elements, not hard-coded). Dirichlet via row/column elimination (or a symmetric penalty-free reduction) so the system remains symmetric. Impose both BCs correctly: modify the RHS by `-K[i,fixed]·Tfixed`.
3. Thomas solver (guard against zero pivots → error). A dense Gaussian reference solver exists only in the test target.
4. Interpolation: `T_fe(x)` within an element by linear shape functions. Midpoint values per element.
5. Exact: `T(x)=100-80x/L+q/(2k)x(L-x)`, evaluated by one function. Errors: nodal max (reported separately for insight) and **midpoint max using FE interpolation**. Convergence: observed order `log2(e_N / e_2N)`.
6. A `solveBatchCase` hook (pure function: problem+N → temperatures) to be reused by P10, with no allocation shared between calls.
**Tests:** Positive: N=1 and N=2 hand-computed K, f, T; midpoint max errors for N=10/20/40/80 within 1e-9 of 1.25, 0.3125, 0.078125, 0.01953125 (the derived values, expected to be confirmed here); BCs satisfied to 1e-12; Thomas vs dense for N=10/40 agree 1e-10. Observed order ≈ 2.0. Negative: k=0, k<0, A<0, N=0, NaN, T0 > 1e308. Edge: T0==TL; q=0 (linear profile, error≈0); N=1; N=1,000,000 (record time, no stack overflow).
**Challenge test (for the AI log):** deliberately ask the assistant, in a separate chat, to state the expected nodal error and the midpoint error, then compare with your code. Record whether its claim was correct. Do not fabricate a mistake.
**Debugging insight:** record the one you actually encounter (e.g. nodal error ≈ 1e-13 despite O(h²) midpoint error and why) in `docs/DEBUG_INSIGHT.md`, only if it truly happened.
**Acceptance:** every test passes; the midpoint expectations are *confirmed by the program output*, not assumed.
**Stop if:** midpoint errors don't reproduce, or nodal-only error is used in any acceptance.

### PROMPT 9: Thermal API, Thermal Analysis view and CSV export (Stage 5/9)
**Requirements:**
1. `POST /api/thermal/solve` with the element list (default `[10,20,40,80]`; validate 1–4096 entries of positive ints within limits) and optional problem overrides (validated). Response: per mesh: node x, nodal T, element midpoints x, FE midpoint T, exact midpoint T, per-midpoint error, max midpoint error, max nodal error, plus a dense exact curve (e.g. 401 points) and the observed orders.
2. `POST /api/export/thermal` → CSV. Columns: `N,element,x_mid_m,T_fe_mid_C,T_exact_mid_C,abs_error_C` (and a second nodal CSV with `N,node,x_m,T_fe_C,T_exact_C`). Save the default-run CSVs to `results/csv/`.
3. UI: parameter form (defaults shown, ranges, units), Run button, status, results: Plotly line chart (T [°C] vs x [m]) with the exact curve plus a trace per N (toggleable), a second chart of max midpoint error vs N (log–log, with slope), a table (N, h, max midpoint error, nodal error, observed order), an "Assumptions and boundary conditions" panel (constant properties, insulated lateral surface, units, Dirichlet BCs, governing equation), PNG export of both plots, CSV download.
4. Save plot PNGs into `results/plots/` (manually via the UI export button, and list the filenames in TESTING.md).
**Tests:** Positive: endpoint numbers match the P8 core tests. Negative: N=0, negative k, an empty list, 10,000 entries → clear envelope errors displayed inline. Edge: only one mesh (no convergence order shown, with an explanation); exported CSV re-parsed by a test and compared with the response.
**Acceptance:** the UI plot and the table agree with `results/csv`; plot axes labelled with units; the debugging insight appears in the UI "Notes" panel only if it truly occurred.
**Stop if:** any displayed number is computed in TypeScript (except unit formatting).

### PROMPT 10: Parallel batch engine (Stage 6)
**Requirements:**
1. `BatchSpec{caseCount, meshElements, seed}` with validation. Generator uses `std::mt19937_64` and explicit `uniform_real_distribution` draws made by a *portable* transformation (document the choice, since distributions are not guaranteed identical across standard libraries, so reproducibility is stated for the same toolchain, and report this limitation). Varied positive k (e.g. 1–100) and q (e.g. 1e3–1e5); other parameters fixed as in the assignment. Generate the full case list first, before timing.
2. `ThreadPool`: bounded, fixed workers, task queue, `submit`/`wait`, destructor joins, no detached threads. Parallel runner partitions case indices in chunks (or an atomic next-index counter); results written to a pre-sized vector by index, so deterministic ordering with no result lock. Cancellation checked between cases.
3. `SerialRunner` and `ParallelRunner(p)` call the **same** `solveBatchCase`. Output: a per-case temperature vector (or a compact checksum plus max-diff comparison utility). `maxAbsDifference(serial, parallel)`.
**Tests:** Positive: same seed → identical case list; serial vs parallel(1,2,3,4) max diff ≤ 1e-8 (expect 0 exactly, and report that); order preserved (case i result equals serial case i). Negative: caseCount=0, mesh=0, negative workers, workers > cap → clamped or an error per documented rule. Edge: caseCount < workers; caseCount=1; worker task throwing → propagated as an error, the pool remains usable; cancel mid-run returns `cancelled`, no partial result published; ThreadSanitizer-equivalent check: run with Clang/ASan+TSan if available, otherwise stress test 200 repetitions and document the limitation honestly.
**Acceptance:** all pass; pool has no leaked threads (test thread count before/after).
**Stop if:** results depend on scheduling.

### PROMPT 11: Benchmark harness, system info and performance API (Stage 6)
**Requirements:**
1. `SystemInfo`: CPU brand string (`__cpuid` leaves), physical cores and logical processors (`GetLogicalProcessorInformationEx`), total RAM (`GlobalMemoryStatusEx`), OS version, plus compiler and optimisation flags from `build_info.hpp`. Missing values are reported as "unavailable", never guessed.
2. `Benchmark`: workers list = `{1,2,3,4}` capped to logical CPUs. For each: 1 warm-up run (not counted), 5 measured repetitions with `steady_clock` around the compute only (case generation, JSON, file I/O outside the timer). Median, min, max. `speedup = T1/Tp`, `efficiency = speedup/p`. Verify the max abs difference vs serial for every p, and fail the job if > 1e-8. If logical CPUs == 1: run the parallel path with 2 workers for demonstration and set `speedupAssessable=false` with a disclosure string.
3. Job API: `POST /api/jobs/performance` → progress per repetition and cancel. The result contains system info, workload, per-p times, stats, differences, `launchToReadyMs` (separate field).
4. `POST /api/export/benchmark` → CSV (raw repetition times) and JSON; the default-run outputs saved to `results/benchmarks/`. Default workload sized so serial median ≥ 0.5 s on your machine (record the actual choice and measurement).
**Tests:** Positive: median/speedup/efficiency on injected synthetic timings (unit-test the statistics with known numbers). Negative: maxWorkers=0 or 17, case count huge (limit error). Edge: logical CPUs mocked as 1, 2, 3 via an injectable provider; cancel mid-benchmark; two benchmark jobs started concurrently (the second is rejected or queued per documented rule, so timings aren't contaminated).
**Acceptance:** a real run on your machine produces files in `results/benchmarks/`; record real numbers only.
**Stop if:** timings include generation or serialization.

### PROMPT 12: Performance view (Stage 6/9)
**Requirements:**
1. Form: case count, mesh elements, seed, max workers (validated, units, defaults), Run/Cancel, a progress bar showing the current worker count and repetition.
2. Results: system info table, workload table, timing table (p, five times, median, speedup, efficiency, max diff), Plotly charts (median time vs p; speedup vs p with ideal line; efficiency vs p), pass/fail agreement badge, launch-to-ready shown in a separate card, the disclosure banner when the CPU cap applies, an "Overhead and limitations" text panel (thread creation, memory bandwidth, hyper-threading, small-workload effects), and CSV/JSON/PNG exports.
3. The UI remains interactive during the run (verify by switching views mid-run, with the job preserved).
**Tests:** Positive: render with a real result fixture captured from a P11 run (labelled as a fixture). Negative: backend error → ErrorPanel. Edge: cancel then rerun; navigating away and back shows the same job.
**Acceptance:** a full run from the UI; the exported CSV equals the backend result; no hard-coded numbers in the UI.
**Stop if:** the analysis text claims speedup that the data does not show. Text must be conditional on measured values or neutral.

### PROMPT 13: Hardware mock adapters (Stage 7)
**Requirements:**
1. `IDeviceAdapter`: `discover()`, `capabilities()`, `connect()`, `disconnect()`, `readTelemetry()`, `state()`. Three mocks: `MockMicrocontrollerBoard` (generic 3.3 V Cortex-M-class board, UART/SWD-JTAG header capabilities; telemetry: supply voltage, temp, uptime), `MockFpgaBoard` (config-via-JTAG, bank voltage, core voltage, config-done flag), `MockJtagAdapter` (debug adapter: VTref sense, TCK frequency, target-attached flag; telemetry: VTref, TCK). Every message carries `simulated:true` and the label "SIMULATED".
2. State machine: Disconnected → Connected → (Disconnected). Illegal transitions return `E_INVALID_STATE`. Deterministic fault injection: the first `readTelemetry` after `armTimeout()` (or a configurable scripted schedule, not randomness) returns `E_DEVICE_TIMEOUT`; the next call succeeds, which is the recoverable-failure demonstration. A retry policy lives in the registry, not in the adapter.
3. `DeviceRegistry` with thread-safe access (mutex), a stable device ordering, and the telemetry timestamps taken from an injectable clock for deterministic tests.
4. Endpoints per Part 3, plus `POST /api/hardware/{id}/inject-timeout`.
**Tests:** Positive: discover returns exactly 3 devices; capabilities fields non-empty; connect → telemetry → disconnect. Negative: telemetry while disconnected; double connect; unknown id → 404. Edge: timeout then immediate success; timeout on each device type; concurrent telemetry calls on one device from 8 threads (no race); the JTAG adapter reports "no target" when not attached and connecting is still allowed (debug adapter semantics).
**Acceptance:** all pass; no mock returns data that implies real hardware.
**Stop if:** any simulated behaviour is not labelled.

### PROMPT 14: Hardware view and embedded design package (Stage 7/9)
**Requirements:**
1. UI: three device cards (discover, capabilities table, Connect/Disconnect, Read telemetry, Inject timeout) with a persistent "SIMULATED, no physical hardware" banner. Telemetry log pane with timestamps. The timeout path shows the error, then a Retry button succeeds.
2. `docs/diagrams/embedded_block_diagram.svg` (hand-authored, labelled: Host PC, USB, MCU board, FPGA board, JTAG adapter, target headers, power rails, grounds, voltage domains) and shown in a "Design" tab.
3. `docs/EMBEDDED_DESIGN.md` plus the in-UI table: interface/pin-voltage table with columns: Interface, Signals, Direction/owner, Voltage domain, Level shifting, Connector, Notes. Must cover: host power (USB 5 V), 3.3 V MCU domain, FPGA I/O bank and core domains (all marked as **assumptions** because the exact boards are unspecified), common ground and single-point reference, JTAG signals TCK/TMS/TDI (adapter → target), TDO (target → adapter), VTref (target → adapter, sets adapter I/O level), optional TRST/SRST, a level-shifting requirement when VTref differs, connector assumptions (different headers on the three devices, so no assumption they match), host comms (USB, UART, etc.), and device-specific pinout caveats.
4. Explanation: a JTAG adapter is a debug/programming interface that attaches to a target's TAP, not a compute board, and boards do not share connector, voltage or protocol.
5. Safe integration plan (power-off wiring, verify VTref, ground first, current limits, staged bring-up, ESD, never exceed absolute max) and limitations.
**Tests:** Positive: the UI flow for all three devices. Negative: telemetry before connect shows the envelope hint. Edge: a timeout on the JTAG adapter and recovery.
**Acceptance:** diagram and table reviewed against the PDF's list item by item (checklist in TESTING.md); screenshots captured.
**Stop if:** any pin/voltage claim is presented as fact instead of an assumption.

### PROMPT 15: Backpropagation core in C++ (Stage 8)
**Requirements:**
1. `Model{w,b,v,c}`; `forward(x)`: `z=wx+b`, `h=tanh(z)`, `ŷ=vh+c`, `J=0.5(ŷ-y)²`. Defaults x=1, y=2, w=0.5, b=0, v=1, c=0, lr=0.1.
2. Analytic gradients (no autodiff, no generic graph): `e=ŷ-y`, `dJ/dc=e`, `dJ/dv=e·h`, `dJ/dh=e·v`, `dz=dJ/dh·(1-h²)`, `dJ/dw=dz·x`, `dJ/db=dz`.
3. Central differences with ε=1e-6 on each parameter perturbing the *loss function*; absolute error per parameter ≤ 1e-5, else the result is flagged failing.
4. One **simultaneous** update (all four use gradients from the original parameters, applied together). Then 100 subsequent training steps with the same lr, loss recorded at each step (step 0 = initial, then 100 after).
5. Tests compute the expected values independently, hand-calculated: `z=0.5`, `h=tanh(0.5)≈0.4621172`, `ŷ≈0.4621172`, `e≈-1.5378828`, `J≈1.18254`, `dJ/dv≈-0.7107`, `dJ/dc≈-1.5379`, `dJ/dw=dJ/db≈-1.2095`, updated `w≈0.6209`, `b≈0.1209`, `v≈1.0711`, `c≈0.1538`. These are my approximate arithmetic checks; treat the program output as authoritative at full precision and tell me if any differs materially.
6. `POST /api/backprop/run` returning forward values, analytic gradients, numeric gradients, errors, pass flag, updated parameters, loss series, and a CSV export (`results/csv/backprop_loss.csv`).
**Tests:** Positive: gradients match the hand values to 1e-4 and the numerical ones to 1e-5; loss decreases monotonically for 100 steps at lr=0.1. Negative: NaN inputs, lr ≤ 0, steps = 0 or > limit → errors. Edge: x=0 (dw=0), very large |w| (tanh saturation, gradient vanishing, no NaN), ε too large vs too small shows degraded error (documented, optional).
**Acceptance:** all four gradient errors ≤ 1e-5 measured; the update is demonstrably simultaneous (a test where sequential updating would give a different result).
**Stop if:** any gradient fails the tolerance.

### PROMPT 16: Backpropagation view and handwritten appendix support (Stage 8/9)
**Requirements:**
1. UI: input form (defaults shown, with the model equations rendered as plain text/unicode, no math library needed), Run, a table of forward values, a gradient table (analytic, numerical, abs error, pass/fail against 1e-5), the update table (old → new), a Plotly loss chart (loss vs step, axis labels, 100 steps), an explanation panel about gradient direction and the observed loss trend written to match the actual output (static text must not assert numbers that the data may not show), and CSV/PNG export.
2. Create `docs/handwritten/README.md` stating: the derivation must be handwritten and scanned by you (≤2 pages), what to include (forward pass, each chain-rule step for w, b, v, c, numeric values, one simultaneous update, gradient-direction note), and a **numeric self-check table** populated from the program output *after* you do it by hand. I will not supply the derivation.
**Tests:** Positive: UI shows values equal to the API. Negative: invalid learning rate → inline error. Edge: lr large enough to oscillate (e.g. 5) shows the loss chart correctly without crashing.
**Acceptance:** results CSV and plot saved in `results/`; the handwritten scan placeholder instructions exist.
**Stop if:** you are asked to write the handwritten derivation itself.

### PROMPT 17: UI consistency, accessibility and responsiveness pass (Stage 9)
**Requirements:**
1. Audit all five views against one design system: spacing scale, type scale, colour tokens, border/radius conventions (minimal rounding), table styles, form layouts. Remove any decoration without function.
2. Consistent status model (idle/running/success/error/cancelled) with distinct visual treatment for inputs, status, results, errors.
3. Responsive down to ~1024 px width and usable at 768 px (collapsible sidebar). Keyboard navigation, visible focus, labelled controls, `aria-live` for status/errors, adequate contrast (check, don't assume), no colour-only meaning in plots and legends.
4. Loading and empty states everywhere; every export button works; every control has real behaviour (remove any that doesn't).
5. Check there is no AI-generated runtime content, and no network use.
**Tests:** Positive: a manual checklist per view (recorded). Negative: browser zoom 200%, a long error message, an overflowing table. Edge: window resize during the 3D view and plots.
**Acceptance:** `npm run build` succeeds with no TS errors/warnings; a checklist is completed with screenshots at 1440, 1024, 768 px.
**Stop if:** the pass changes any backend contract.

### PROMPT 18: Integration, robustness and quality gates (Stage 10)
**Requirements:**
1. End-to-end API tests (C++ test executable or scripted `curl`-less client in C++, since tests aren't backend features but must not use Python) running the real server on an ephemeral port: all endpoints, positive and negative.
2. Fuzz-style malformed inputs for both parsers (truncate at every byte of a small valid file, random byte flips with a fixed seed): must never crash or hang, must return a structured error or a valid parse.
3. Size-limit tests recorded as measurements (largest STL/CSV accepted, time, peak memory), written into `LIMITATIONS.md` and the in-UI limits display.
4. Startup/shutdown tests: start/stop 50×; stop with running jobs; double-start of the server on the same port (port 0 avoids it); orphan-process check.
5. Thread safety: repeat the pool, job and hardware concurrency tests 200×; run AddressSanitizer/UBSan builds if your toolchain supports them (state precisely what was and wasn't run).
6. Full regression run; record all results in `TESTING.md` with the test inventory mapped to requirement IDs.
**Acceptance:** every module has unit, negative and edge tests; zero flaky tests across 50 repeated full runs (record real counts); the traceability table in `TESTING.md` links each requirement to a passing test.
**Stop if:** any test is skipped or relaxed to pass.

### PROMPT 19: Launcher, packaging and offline verification (Stage 11)
**Requirements:**
1. `src/launcher`: Win32 GUI-subsystem exe (no console window), single-instance mutex, records launch timestamp, creates a Job Object with kill-on-close, spawns `app\workbench_server.exe --port 0 --launch-ts … --web-root app\web` hidden, reads the port line with a timeout, polls `/api/health` (timeout 15 s), then opens `msedge --app=` (if found) else the default browser via `ShellExecute`. On failure shows a native message box with the cause and log path. Uses relative paths from its own module location, never the working directory.
2. `packaging/windows/build_release.bat`: configure, release build, `npm ci && npm run build` (developer machine only), stage the `dist/` layout and `src/`, copy the docs, data, tests, results, the `.vscode/` config, `THIRD_PARTY.md`, git bundle and commit log, then zip as `CandidateID_Workbench.zip` (parameterised ID).
3. `.vscode/launch.json` with a configuration "Run packaged Workbench" using `"type":"node-terminal"` (no extension needed) or `cppvsdbg` if you choose that (document the extension prerequisite in `editor-config/README.md`) that starts `${workspaceFolder}/Launch.exe`; `tasks.json` as a fallback. Test the config in both VS Code and Antigravity and record the results.
4. Verification: `dumpbin /dependents` on both exes (no non-system DLLs); run from a path with spaces and from a different working directory; run with the network disabled; run as a standard user; check for leftover processes after closing; measure launch-to-ready (5 runs, recorded).
5. A clean-machine procedure (fresh Windows 11 VM or a second account without dev tools) is in `docs/CLEAN_MACHINE_TEST.md` for you to execute. I cannot claim it passed, so record the result only when you've actually run it.
**Acceptance:** double-clicking `Launch.exe` from the extracted ZIP opens the working app in ≤ 15 s; all views functional offline; no processes remain after closing.
**Stop if:** any runtime DLL outside the system set is required, or anything needs a terminal.

### PROMPT 20: Documentation, report and final submission (Stage 12)
**Requirements:**
1. `README.md`: prerequisites (none for evaluators), one-click steps, architecture, supported formats, build/test steps, limitations, SmartScreen note, editor run configuration.
2. `docs/REPORT.md` (≤ 6 pages when exported to PDF, excluding the handwritten appendix): design choices, FEA verification (the midpoint table, convergence, the debugging insight), performance results (the measured table and the honest overhead analysis), embedded design, test outcomes, limitations, the actual effort hours. Every number must be traceable to a `results/` file.
3. `docs/AI_USE_LOG.md`: real tools/models, representative prompts, outputs, your edits, validation evidence, and at least one genuinely incorrect or unsuitable suggestion, or the documented challenge test from P8 with its true outcome.
4. `docs/OWNERSHIP.md`: original vs reused (vendored libraries, snippets) work.
5. Final labelled plots in `results/plots/`, raw CSVs in `results/csv/`, benchmark files in `results/benchmarks/`.
6. Git: create the bundle (`git bundle create`) and the readable `commit_log.txt`; verify that `git clone` from the bundle builds.
7. Demo evidence: a 3–5 min video script or a timestamped screenshot walkthrough with logs (`docs/evidence/`).
8. Run the final checklist (Part 8) and fill in the results truthfully.
**Acceptance:** the ZIP extracts, launches, and every checklist item is either evidenced or listed as a known limitation.
**Stop if:** any claim in the report lacks a source file.

---

# PART 7: PERSISTENT CONTEXT STRATEGY

**Mechanism:** `PROJECT_SPEC.md` (what), `ARCHITECTURE.md` (how), `PROJECT_STATE.md` (where we are), `DECISIONS.md` (why), `TESTING.md` (proof). The PREAMBLE forces every conversation to read these first and to inspect the real repo. The FOOTER forces a state update and a commit at the end. `PROJECT_STATE.md` always contains: the phase, completed prompts, files, working and missing features, tests with real results, bugs, blockers and **the exact text of the next prompt**. So any new chat can resume from the files alone. Chat history is never a dependency. `git log` provides the independent record of the true state; if the state file and git disagree, git plus the real test run wins.

### CONTINUE PROJECT (paste into a new chat)
```
CONTINUE PROJECT: Unified Engineering Workbench (PASS SCIENCE WORKS internship).
Do not rely on earlier chat history. First read, in order:
docs/PROJECT_SPEC.md, docs/ARCHITECTURE.md, docs/PROJECT_STATE.md, docs/DECISIONS.md.
Then inspect the real repo (directory listing, `git log --oneline -15`, `git status`)
and run the existing test suite. Report: (1) last completed prompt, (2) next pending
prompt, (3) any discrepancy between the docs, repo and test results. Resolve
discrepancies before coding. Then execute the next prompt in
docs/IMPLEMENTATION_PLAN.md exactly, following its acceptance criteria, update
PROJECT_STATE.md, TESTING.md and AI_USE_LOG.md with real results only, commit, and
end with a handoff summary. Never mark incomplete work as done; never invent results.
```

---

# PART 8: FINAL VERIFICATION CHECKLIST (100-point rubric)

Tick only items with evidence you can point to.

**Interface (8)**
- [ ] Five views, consistent layout, status always visible
- [ ] Validation messages are inline, specific and actionable
- [ ] Plots have titles, labelled axes with units, legends
- [ ] 1024 px and 768 px layouts checked

**C++ backend and testing (12), gate ≥ 6**
- [ ] All computation, parsing and threading in C++
- [ ] Unit + integration + negative + edge + numerical tests, all pass
- [ ] Errors are structured, with no crashes on malformed input
- [ ] Core builds and tests without the HTTP library

**Folder architecture (6)**
- [ ] `src/` layout as required, relative paths only, dependencies vendored and recorded

**Code readability (6)**
- [ ] Small functions, consistent naming, only non-obvious comments, no TODO/FIXME

**Version control (5)**
- [ ] Meaningful commits per prompt, `.gitignore` clean, bundle plus readable log, `THIRD_PARTY.md`

**Thermal FEA (14), gate ≥ 7**
- [ ] Ke, fe, assembly, both Dirichlet BCs correct (6)
- [ ] Midpoint-interpolation errors for N=10/20/40/80, convergence order ≈ 2, CSV, plots (5)
- [ ] Assumptions, units, BCs, debugging insight, learning explanation (3)

**CAD/ANSYS visualization (10)**
- [ ] ASCII STL and CSV both load via C++
- [ ] Rotate/pan/zoom/fit, legend with °C, node inspection
- [ ] Malformed, non-finite and empty files handled with explanations; limits measured
- [ ] Samples with provenance; CSV labelled SYNTHETIC

**Multicore (10)**
- [ ] Real pool/threads on independent solves (4)
- [ ] Fixed seed, warm-up, 5 reps, median, speedup, efficiency, system info, diff ≤ 1e-8, launch-to-ready separate (4)
- [ ] Honest overhead and limits analysis, single-CPU disclosure if applicable (2)

**Embedded (8)**
- [ ] Block diagram plus pin/voltage table with explicit assumptions, JTAG explanation, integration plan (4)
- [ ] Three mocks working in UI, timeout recovery, simulated labelling (4)

**Backpropagation (7)**
- [ ] Handwritten derivation scanned, ≤ 2 pages, you can defend it (4)
- [ ] C++ gradient check ≤ 1e-5 and loss plot with explanation (3)

**AI judgement (8)**
- [ ] Prompts, iterations, verification, one genuine incorrect suggestion or challenge test, nothing fabricated

**One-click and report (6), gate ≥ 3**
- [ ] Clean-machine double-click launch, no terminal/admin/install/internet (3)
- [ ] Editor run configuration works
- [ ] Report ≤ 6 pages, evidence traceable, limitations stated (3)
- [ ] ZIP named `CandidateID_Workbench.zip`, includes video/screenshots, README, ownership statement

**Gate summary:** working C++ backend ✔/✘ · standalone launch ✔/✘ · backend ≥ 6/12 ✔/✘ · FEA ≥ 7/14 ✔/✘ · delivery ≥ 3/6 ✔/✘

---

**Caveats**
- Library versions, the measured size limits and all benchmark numbers must come from your actual runs. I did not invent any.
- The handwritten backprop derivation, the clean-machine launch test and the AI-log "incorrect suggestion" must be your own genuine work and observations.

When ready, send me Prompt 1's output and `PROJECT_STATE.md` after you run it, and I'll review before you proceed.