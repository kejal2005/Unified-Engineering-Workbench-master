# Build and Packaging Guide

## 1. Prerequisites
- **Operating System**: Windows 10 / 11 x64
- **Toolchain**: Microsoft Visual Studio 2022 (MSVC v143 or newer) with Desktop development with C++
- **Build System**: CMake $\ge 3.20$, Ninja build generator
- **Node.js Environment**: Node.js $\ge 18.x$ and npm (required only during developer packaging, not for end-user execution)

## 2. Developer Build Instructions

### C++ Backend Compilation
```powershell
cmake --preset release
cmake --build --preset release
```

### Frontend Bundle Compilation
```powershell
cd src/frontend
npm ci
npm run build
```

## 3. Packaging Pipeline
The automated packaging script `packaging/windows/build_release.bat` executes the release assembly:
1. Performs clean release build of `Launch.exe` and `workbench_server.exe`.
2. Compiles production React assets into `dist/` and stages them into `app/web/`.
3. Verifies static runtime linkage via `dumpbin /dependents` to confirm absence of non-system DLL dependencies.
4. Validates offline execution by running loopback health check.
5. Assembles directory payload into `CandidateID_Workbench.zip`.

