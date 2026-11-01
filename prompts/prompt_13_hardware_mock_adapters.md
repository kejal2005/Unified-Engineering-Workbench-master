# PROMPT 13: Hardware mock adapters (Stage 7)

## Requirements
1. `IDeviceAdapter`: `discover()`, `capabilities()`, `connect()`, `disconnect()`, `readTelemetry()`, `state()`. Three mocks: `MockMicrocontrollerBoard` (generic 3.3 V Cortex-M-class board, UART/SWD-JTAG header capabilities; telemetry: supply voltage, temp, uptime), `MockFpgaBoard` (config-via-JTAG, bank voltage, core voltage, config-done flag), `MockJtagAdapter` (debug adapter: VTref sense, TCK frequency, target-attached flag; telemetry: VTref, TCK). Every message carries `simulated:true` and the label "SIMULATED".
2. State machine: Disconnected -> Connected -> (Disconnected). Illegal transitions return `E_INVALID_STATE`. Deterministic fault injection: the first `readTelemetry` after `armTimeout()` returns `E_DEVICE_TIMEOUT`; the next call succeeds (recoverable failure demonstration). Retry policy in registry, not adapter.
3. `DeviceRegistry` with thread-safe access (mutex), stable device ordering, telemetry timestamps taken from injectable clock.
4. Endpoints per Part 3, plus `POST /api/hardware/{id}/inject-timeout`.

## Tests
- Positive: discover returns exactly 3 devices; capabilities fields non-empty; connect -> telemetry -> disconnect.
- Negative: telemetry while disconnected; double connect; unknown id -> 404.
- Edge: timeout then immediate success; timeout on each device type; concurrent telemetry calls on one device from 8 threads; JTAG adapter reports "no target" when not attached and connecting is still allowed.

## Acceptance
All pass; no mock returns data that implies real hardware.

## Stop If
Any simulated behaviour is not labelled.

