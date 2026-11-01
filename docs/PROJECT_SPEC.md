# Project Specification: Unified Engineering Workbench

## 1. System Overview
The Unified Engineering Workbench is an offline Windows desktop engineering application combining CAD and simulation data visualization, 1D thermal finite element analysis (FEA), parallel benchmark evaluation, embedded hardware interface modeling, and backpropagation mechanics.

The application architecture separates presentation (React + TypeScript + Three.js + Plotly) from computational processing (C++17 backend). A native Win32 launcher (`Launch.exe`) provides single-click startup on clean Windows 11 x64 systems without requiring administrator rights, terminal interaction, preinstalled runtimes, or runtime network access.

## 2. Requirement Baseline

### Category A: Interface and Launcher
- **A1**: Single desktop interface containing five functional views: Geometry, Thermal Analysis, Multicore Performance, Hardware Interface, and Backpropagation.
- **A2**: Native file selection with client-side path abstraction and server-side byte streaming.
- **A3**: Real-time parameter validation with actionable error envelopes before compute dispatch.
- **A4**: Progress and status feedback for asynchronous jobs with cooperative cancellation.
- **A5**: Responsive UI during heavy numerical computations via non-blocking polling architecture.
- **A6**: Native data export (CSV format) for simulation, performance, and gradient metrics.
- **A7**: Decoupled C++ computational core independent of HTTP transport and UI rendering layers.
- **A8**: Standalone `Launch.exe` binary initiating the application on clean Windows 11 x64.
- **A9**: Editor run configuration (`.vscode/launch.json`) targeting the packaged application.
- **A10**: Documented offline CMake source-build procedures.

### Category B: CAD and Simulation Visualization
- **B1**: ASCII STL parser validating header, facets, normal vectors, and loop boundaries.
- **B2**: Three.js 3D viewport providing orbit, pan, zoom, coordinate axes, and fit-to-view.
- **B3**: Nodal CSV parser enforcing exact schema `node_id,x_m,y_m,z_m,temperature_C`.
- **B4**: Colored point cloud rendering with scalar temperature field and legend in degrees Celsius (°C).
- **B5**: Interactive node inspection identifying node ID, coordinates, and local temperature.
- **B6**: Rejection of malformed, truncated, or non-finite numerical entries (NaN, Inf).
- **B7**: Clear diagnostics explaining empty or header-only files.
- **B8**: Tested and documented file size limits for geometric and tabular imports.
- **B9**: Bundled sample files (`sample_part.stl` and `sample_nodal_temperature_SYNTHETIC.csv`) with full provenance documentation.

### Category C: Thermal Finite Element Analysis
- **C1**: Steady-state 1D heat conduction governed by `-k * d^2T/dx^2 = q`.
- **C2**: Standard problem parameters: Length $L = 1.0\text{ m}$, Area $A = 1.0 \times 10^{-4}\text{ m}^2$, Conductivity $k = 10.0\text{ W/(m}\cdot\text{K)}$, Heat Source $q = 10000.0\text{ W/m}^3$, Boundary Conditions $T(0) = 100.0^\circ\text{C}$, $T(L) = 20.0^\circ\text{C}$.
- **C3**: Two-node linear element formulation: element stiffness matrix $K_e = \frac{kA}{h}\begin{bmatrix}1 & -1 \\ -1 & 1\end{bmatrix}$, element force vector $f_e = \frac{qAh}{2}\begin{bmatrix}1 \\ 1\end{bmatrix}$.
- **C4**: Global system assembly into tridiagonal format and exact Dirichlet boundary imposition.
- **C5**: Linear Thomas algorithm solver ($O(N)$ complexity) with zero-pivot protection.
- **C6**: Exact analytical solution: $T(x) = 100 - 80\frac{x}{L} + \frac{q}{2k}x(L - x)$.
- **C7**: Evaluation of element midpoint values via linear finite element shape function interpolation: $T_{fe}(x_{mid}) = 0.5(T_i + T_{i+1})$.
- **C8**: Midpoint error convergence verification for mesh discretizations $N \in \{10, 20, 40, 80\}$.
  - Theoretical midpoint maximum error: $e_{mid} = \frac{q h^2}{8k}$.
  - Expected maximum errors:
    - $N = 10$ ($h = 0.1\text{ m}$): $1.25^\circ\text{C}$
    - $N = 20$ ($h = 0.05\text{ m}$): $0.3125^\circ\text{C}$
    - $N = 40$ ($h = 0.025\text{ m}$): $0.078125^\circ\text{C}$
    - $N = 80$ ($h = 0.0125\text{ m}$): $0.01953125^\circ\text{C}$
  - Observed order of convergence: $p = \log_2(e_N / e_{2N}) \approx 2.0$.
- **C9**: Tabular and graphic comparison of FE results versus analytical solution.
- **C10**: Complete CSV export of nodal and midpoint values with associated errors.

### Category D: Multicore Performance Benchmark
- **D1**: Parametric batch solve generator with fixed-seed pseudorandom generation (`std::mt19937_64`).
- **D2**: Varied thermal conductivity $k \in [1, 100]\text{ W/(m}\cdot\text{K)}$ and volumetric heat $q \in [1000, 100000]\text{ W/m}^3$.
- **D3**: Deterministic output ordering matching serial execution sequence.
- **D4**: Thread pool worker execution supporting thread concurrency $p \in \{1, 2, 3, 4\}$, capped at logical CPU count.
- **D5**: Execution harness running 1 warm-up pass followed by 5 measured repetitions per worker count.
- **D6**: Statistical reporting of median compute time, minimum time, maximum time, speedup ($S_p = T_1 / T_p$), and parallel efficiency ($E_p = S_p / p$).
- **D7**: System telemetry capture: CPU model, physical core count, logical core count, total RAM, compiler identification, and build flags.
- **D8**: Numerical verification confirming serial and parallel output equivalence within absolute tolerance $\le 10^{-8\circ}\text{C}$.
- **D9**: Separate reporting of server launch-to-ready latency.
- **D10**: Single-CPU execution disclosure if executed on single-core host environments.

### Category E: Embedded Hardware Interface Modeling
- **E1**: System block diagram detailing Host PC, USB interconnects, microcontroller target, FPGA target, and JTAG hardware debug probe.
- **E2**: Interface voltage and signal reference table documenting supply rails, ground references, logic levels, level shifting, connectors, and interface directions.
- **E3**: Technical documentation detailing JTAG adapter operation as an external debug TAP controller rather than a peripheral compute node.
- **E4**: C++ device adapter abstractions (`IDeviceAdapter`) for Microcontroller, FPGA, and JTAG debug units.
- **E5**: Explicit labeling of all telemetry and state responses as simulated data.
- **E6**: Deterministic fault injection mechanism demonstrating recoverable communication timeouts.
- **E7**: Documented hardware bring-up safety plan, electrical limits, and isolation boundaries.

### Category F: Backpropagation Numerical Verification
- **F1**: Scalar forward pass evaluation: $z = w \cdot x + b$, $h = \tanh(z)$, $\hat{y} = v \cdot h + c$, $J = \frac{1}{2}(\hat{y} - y)^2$.
- **F2**: Baseline parameters: $x = 1.0$, $y = 2.0$, $w = 0.5$, $b = 0.0$, $v = 1.0$, $c = 0.0$, learning rate $\eta = 0.1$.
- **F3**: Analytical gradient calculation:
  - Error: $e = \hat{y} - y$
  - $\frac{\partial J}{\partial c} = e$
  - $\frac{\partial J}{\partial v} = e \cdot h$
  - $\frac{\partial J}{\partial h} = e \cdot v$
  - $\delta_z = \frac{\partial J}{\partial h} \cdot (1 - h^2)$
  - $\frac{\partial J}{\partial w} = \delta_z \cdot x$
  - $\frac{\partial J}{\partial b} = \delta_z$
- **F4**: Central difference numerical gradient verification with perturbation $\epsilon = 1.0 \times 10^{-6}$:
  - $\frac{\partial J_{num}}{\partial \theta} = \frac{J(\theta + \epsilon) - J(\theta - \epsilon)}{2\epsilon}$
  - Absolute gradient error tolerance: $|\nabla_{analytic} - \nabla_{num}| \le 1.0 \times 10^{-5}$.
- **F5**: Single simultaneous parameter update applying all computed gradients in parallel.
- **F6**: Multi-step training trajectory recording loss decrease across 100 iterations.
- **F7**: Reference check values:
  - $z = 0.500000$
  - $h = \tanh(0.5) \approx 0.462117$
  - $\hat{y} \approx 0.462117$
  - $e \approx -1.537883$
  - $J \approx 1.182541$
  - $\frac{\partial J}{\partial c} \approx -1.537883$
  - $\frac{\partial J}{\partial v} \approx -0.710682$
  - $\frac{\partial J}{\partial w} \approx -1.209503$
  - $\frac{\partial J}{\partial b} \approx -1.209503$
  - Updated $w \approx 0.620950$, $b \approx 0.120950$, $v \approx 1.071068$, $c \approx 0.153788$.
- **F8**: Integration support for supplementary scanned handwritten derivation ($\le 2$ pages).

### Category ZIP: Packaging and Delivery
- **ZIP1**: Packaging into `CandidateID_Workbench.zip`.
- **ZIP2**: Inclusion of executable `Launch.exe`, packaged binaries, full source tree, CMake configurations, test suites, provenance-backed samples, and generated results.
- **ZIP3**: Comprehensive engineering report ($\le 6$ pages) detailing methodology, verification proofs, and performance measurements.
- **ZIP4**: Transparent AI usage log documenting engineering prompts, verification steps, and challenge tests.

## 3. Shortlist Gate Criteria
1. Fully functioning C++ computational core.
2. Unassisted standalone launch on clean Windows 11 x64 systems.
3. C++ backend score $\ge 6 / 12$.
4. Thermal FEA score $\ge 7 / 14$.
5. Delivery and documentation score $\ge 3 / 6$.
6. Cumulative assessment score $\ge 50$ (Interview Band) / $\ge 80$ (Strong Hire Band).

