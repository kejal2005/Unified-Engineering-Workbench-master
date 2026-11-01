# System Boundaries and Limitations

## 1. Architectural Boundaries
- **Network Isolation**: The application strictly binds to `127.0.0.1`. Remote network connections are actively rejected.
- **File System Sandboxing**: The server processes files solely provided via HTTP multipart or direct body streams. Arbitrary server path traversal is prevented.
- **Single Instance Enforcement**: `Launch.exe` enforces a single active execution instance via a named Windows Mutex (`Global\UnifiedEngineeringWorkbenchMutex`).

## 2. Format and Data Scope
- **CAD Geometries**: Supports ASCII STL files up to configured facet limits. Binary STL files are rejected with diagnostic instructions.
- **Simulation Data**: Nodal CSV files require strict column conformance (`node_id,x_m,y_m,z_m,temperature_C`). Missing columns or unparseable floating-point tokens trigger immediate rejection.
- **Physical Hardware**: All hardware devices (Microcontroller, FPGA, JTAG) operate in mock emulation mode. No physical hardware connection is attempted or required.

## 3. Measured Limits Table

| Metric | Target Specification | Tested Operating Limit | Failure Mode |
|---|---|---|---|
| Maximum STL File Size | 50 MB | To be determined in P18 | HTTP 413 Payload Too Large |
| Maximum Facet Count | 500,000 facets | To be determined in P18 | Diagnostic validation error |
| Maximum CSV Rows | 250,000 nodes | To be determined in P18 | HTTP 413 Payload Too Large |
| Max Batch Solve Cases | 50,000 cases | To be determined in P18 | Job validation rejection |

