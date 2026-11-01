# Unified Engineering Workbench

A cross-disciplinary desktop workbench for engineering analysis, combining 3D CAD/ANSYS visualization, 1D steady-state finite element thermal analysis, multicore batch execution benchmarks, embedded device simulation, and verified neural backpropagation mechanics.

## Key Capabilities

- **CAD & Simulation Data Visualization**: High-performance ASCII STL and CSV nodal temperature parsing with Three.js rendering, orbit controls, bounding box fitting, and point picking.
- **Thermal Finite Element Analysis**: 1D steady-state conduction solver using two-node linear elements, tridiagonal Thomas solver ($O(N)$), exact analytical verification, and midpoint interpolation error calculation demonstrating $O(h^2)$ convergence.
- **Multicore Parallel Benchmark**: Parametric batch execution engine with portable random generation, bounded C++ thread pool, deterministic results ordering, and statistical reduction reporting median speedup and efficiency.
- **Embedded Hardware Interface**: Architectural design specification and software mock adapters for Microcontroller, FPGA, and JTAG probe interfaces with fault injection and recovery handling.
- **Backpropagation Engine**: Analytical scalar gradient evaluation verified against central difference numerical gradients ($\epsilon = 1.0 \times 10^{-6}$), simultaneous multi-parameter updates, and 100-step training verification.

## Architecture

```
Launch.exe (Win32) ──► app/workbench_server.exe (C++17) ──► Loopback REST (127.0.0.1:port)
                               │
                               └── serves app/web/ (React + TypeScript UI)
```

The system operates completely offline without network calls, external CDNs, or administrative privileges.

## Directory Layout

```
.
├── CMakeLists.txt
├── CMakePresets.json
├── THIRD_PARTY.md
├── .vscode/
├── editor-config/
├── third_party/
├── src/
│   ├── frontend/
│   ├── backend/
│   └── launcher/
├── data/
│   ├── samples/
│   └── provenance/
├── tests/
├── results/
│   ├── csv/
│   ├── plots/
│   └── benchmarks/
├── docs/
├── scripts/
├── packaging/
└── prompts/
```

## Building from Source

### Prerequisites
- Windows 10/11 x64
- Visual Studio 2022 (MSVC v143+) with C++ Desktop Workload
- CMake 3.20+ and Ninja
- Node.js 18+ (for frontend bundle builds)

### Build Commands
```powershell
cmake --preset release
cmake --build --preset release
ctest --preset release --output-on-failure
```

### Packaging
```powershell
packaging\windows\build_release.bat
```
Produces `CandidateID_Workbench.zip` containing the standalone `Launch.exe`, packaged binaries, docs, test suites, and sample data.

