# System Architecture

## 1. Architectural Overview
The Unified Engineering Workbench uses a decoupled process model: a native C++ computational core served over an internal loopback REST transport, interfaced to a React single-page frontend running inside a local browser container, orchestrated by a native Win32 launcher.

```
+-------------------------------------------------------------------------+
| Browser / WebView Window                                                |
| React 18, TypeScript, Three.js, Plotly.js                               |
| src/frontend/src/                                                       |
+------------------------------------+------------------------------------+
                                     | Loopback HTTP (127.0.0.1:port)
                                     v
+------------------------------------+------------------------------------+
| Transport Layer (C++17)                                                 |
| cpp-httplib, nlohmann/json, JobManager, ErrorMapper, Static Serving     |
| src/backend/src/transport/                                              |
+------------------------------------+------------------------------------+
                                     | Direct C++ API calls
                                     v
+------------------------------------+------------------------------------+
| Computational Core (C++17 Static Library)                               |
| core, geometry, thermal, performance, hardware, backpropagation         |
| src/backend/src/{core,geometry,thermal,performance,hardware,backprop}/  |
+-------------------------------------------------------------------------+
```

## 2. Process Boundary and Lifecycle
1. **Startup**:
   - `Launch.exe` creates a named Windows Mutex to ensure single-instance execution.
   - The launcher spawns `app/workbench_server.exe` as a child process assigned to a Windows Job Object configured with `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`.
   - The backend binds to loopback interface `127.0.0.1` on ephemeral port 0, outputs the selected port string `PORT=<n>` to stdout, and records startup timestamp.
   - `Launch.exe` intercepts the port, polls `GET /api/health` until ready, and launches Microsoft Edge in application mode (`msedge --app=http://127.0.0.1:<port>/`), falling back to the system default browser via `ShellExecuteW`.
2. **Runtime Execution**:
   - The UI issues periodic heartbeats to `POST /api/heartbeat` every 5 seconds.
   - Long-running operations execute asynchronously via the `JobManager` thread pool.
3. **Shutdown**:
   - Closing the browser window terminates client heartbeat transmissions.
   - The backend server monitors an idle heartbeat timeout (default: 15 seconds).
   - Upon timeout expiration, running jobs are signaled via cooperative cancellation tokens, thread pools join, and the server process exits cleanly.
   - If the launcher is closed abruptly, the Windows Job Object automatically terminates the child backend process.

## 3. Communication Protocol and API Contract
All endpoints are bound to `/api/*`. Request and response payloads use UTF-8 JSON encoding.

### API Endpoints
- `GET /api/health`
  - Returns backend readiness status, engine version, limits configuration, and launch-to-ready latency in milliseconds.
- `POST /api/heartbeat`
  - Updates active connection timestamp to maintain server lifecycle. Returns HTTP 204.
- `POST /api/geometry/stl`
  - Accepts raw ASCII STL body.
  - Returns triangle counts, vertex position buffers, facet normals, spatial bounding boxes, and unit specification.
- `POST /api/geometry/nodal-csv`
  - Accepts raw CSV data adhering to `node_id,x_m,y_m,z_m,temperature_C`.
  - Returns node coordinates, scalar temperature arrays, global minimum/maximum values, and units.
- `POST /api/thermal/solve`
  - Accepts discretization array `elements` and optional problem parameter overrides.
  - Returns node distributions, element midpoint coordinates, FE vs analytical temperatures, error distributions, and convergence order metrics.
- `POST /api/jobs/performance`
  - Submits asynchronous benchmark job with specified case count, mesh resolution, random seed, and maximum worker thread count.
  - Returns HTTP 202 with assigned `jobId`.
- `GET /api/jobs/{id}`
  - Polls asynchronous job state (`queued`, `running`, `completed`, `cancelled`, `failed`), progress indicator, and payload.
- `POST /api/jobs/{id}/cancel`
  - Triggers cooperative cancellation flag on the specified job.
- `GET /api/hardware/devices`
  - Enumerates mock hardware devices with capabilities and state indicators.
- `POST /api/hardware/{id}/connect`
  - Connects to the designated mock device.
- `POST /api/hardware/{id}/disconnect`
  - Disconnects from the designated mock device.
- `POST /api/hardware/{id}/telemetry`
  - Reads telemetry payload from connected device.
- `POST /api/hardware/{id}/inject-timeout`
  - Arms a single deterministic communication timeout for recoverable fault demonstration.
- `GET /api/hardware/design`
  - Retrieves reference interface pin and voltage specification table data.
- `POST /api/backprop/run`
  - Evaluates forward network state, analytical gradients, numerical central differences, simultaneous weight updates, and 100-step training trajectory.
- `POST /api/export/{kind}`
  - Generates downloadable CSV streams for thermal, performance, or backpropagation metrics.

### Error Envelope Specification
All failure responses use standardized HTTP status codes and a structured error payload:
```json
{
  "error": {
    "code": "E_SCHEMA_MISMATCH",
    "message": "Header line does not match expected schema: node_id,x_m,y_m,z_m,temperature_C",
    "hint": "Verify column names and verify file is not binary.",
    "details": {
      "line": 1,
      "expected": "node_id,x_m,y_m,z_m,temperature_C",
      "found": "id,x,y,z,temp"
    }
  }
}
```

Standard error codes:
- `E_EMPTY_FILE`: File length is 0 bytes or header only.
- `E_MALFORMED_FILE`: Syntax violation or premature end of file.
- `E_NONFINITE_VALUE`: Encountered NaN, Inf, or unparseable floating-point token.
- `E_SCHEMA_MISMATCH`: CSV header does not match required columns.
- `E_TOO_LARGE`: Payload exceeds configured byte or entity size limits.
- `E_INVALID_PARAMETER`: Numerical parameters out of valid operational range.
- `E_CANCELLED`: Job processing interrupted by client cancel signal.
- `E_DEVICE_TIMEOUT`: Communication timeout with hardware adapter.
- `E_INVALID_STATE`: State machine transition violation.
- `E_NOT_FOUND`: Referenced job or device identifier does not exist.
- `E_INTERNAL`: Unhandled internal server execution fault.

## 4. Concurrency Model and Thread Safety
- The core computational library is free of static mutable state.
- Thermal solver routines operate on independent allocated vectors with no cross-thread memory sharing.
- Multicore batch execution uses a bounded thread pool (`std::vector<std::thread>`).
- Work distribution assigns pre-allocated indexed slots for output results, eliminating mutex contention during result collection.
- Asynchronous job execution tracks progress via `std::atomic<double>` and monitors cancellation via `std::atomic<bool>`.
- Hardware device registry guards state modifications via `std::mutex`.

