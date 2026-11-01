# Verification and Testing Protocol

## 1. Test Architecture
Testing is structured across four layers:
1. **Unit Testing**: GoogleTest fixtures for core data structures, numerical solvers, and parsers.
2. **Numerical Verification**: Direct comparison of solver outputs against closed-form analytical solutions and finite difference approximations.
3. **Integration Testing**: Ephemeral loopback HTTP client calls exercising REST endpoints and job lifecycle state machines.
4. **Stress & Fuzzing**: Malformed payload fuzzing, memory leak tracking, and thread-pool race condition stress tests.

## 2. Requirement Traceability Matrix

| Requirement ID | Module | Verification Method | Pass Criteria |
|---|---|---|---|
| A1-A6 | UI / Shell | Component testing and manual UI audit | Responsive UI, clear error display, functional exports |
| A7 | Core Architecture | Static link analysis | `workbench_core` compiles with zero HTTP/socket dependencies |
| A8 | Launch Automation | Clean machine execution | Native start in $\le 15$ seconds without console or admin rights |
| B1-B8 | Geometry Parsers | Unit and boundary tests | Correct facet count, rejection of NaN/Inf, line-numbered errors |
| C1-C8 | Thermal FEA | Analytical comparison | Midpoint error order $p \approx 2.0$, matches $\frac{q h^2}{8k}$ |
| D1-D8 | Multicore Batch | Thread pool verification | Deterministic result order, serial vs parallel diff $\le 10^{-8}$ |
| E1-E6 | Embedded Mocks | Mock adapter unit tests | State machine transitions, deterministic timeout recovery |
| F1-F6 | Backpropagation | Central difference tests | Analytic vs numerical gradient error $\le 10^{-5}$ |

## 3. Test Execution Commands

```powershell
cmake --preset release
cmake --build --preset release
ctest --preset release --output-on-failure
```

