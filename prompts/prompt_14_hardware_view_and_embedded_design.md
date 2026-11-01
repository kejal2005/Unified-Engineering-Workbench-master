# PROMPT 14: Hardware view and embedded design package (Stage 7/9)

## Requirements
1. UI: three device cards (discover, capabilities table, Connect/Disconnect, Read telemetry, Inject timeout) with persistent "SIMULATED, no physical hardware" banner. Telemetry log pane with timestamps. Timeout path shows error, then Retry button succeeds.
2. `docs/diagrams/embedded_block_diagram.svg` (Host PC, USB, MCU board, FPGA board, JTAG adapter, target headers, power rails, grounds, voltage domains) and shown in "Design" tab.
3. `docs/EMBEDDED_DESIGN.md` plus in-UI table: interface/pin-voltage table (Interface, Signals, Direction/owner, Voltage domain, Level shifting, Connector, Notes). Must cover: host power (USB 5 V), 3.3 V MCU domain, FPGA I/O bank and core domains (all marked as assumptions), common ground, JTAG signals TCK/TMS/TDI/TDO/VTref, level-shifting requirement, connector assumptions, host comms, pinout caveats.
4. Explanation: JTAG adapter is a debug/programming interface that attaches to a target TAP, not a compute board, and boards do not share connector, voltage or protocol.
5. Safe integration plan (power-off wiring, verify VTref, ground first, current limits, staged bring-up, ESD, never exceed absolute max) and limitations.

## Tests
- Positive: UI flow for all three devices.
- Negative: telemetry before connect shows envelope hint.
- Edge: timeout on JTAG adapter and recovery.

## Acceptance
Diagram and table reviewed against PDF list item by item (checklist in TESTING.md); screenshots captured.

## Stop If
Any pin/voltage claim is presented as fact instead of an assumption.

