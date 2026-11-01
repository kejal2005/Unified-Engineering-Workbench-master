# Embedded System Hardware Design Specification

## 1. System Block Diagram
```
+---------------------------------------------------------------------------------+
| Host Workstation                                                                |
+-------+-----------------------------+-----------------------------+-------------+
        | USB 2.0 (5V VBUS / D+ / D-) | USB 2.0 (5V VBUS / D+ / D-) | USB 2.0
        v                             v                             v
+-------+-------------+       +-------+-------------+       +-------+-------------+
| JTAG Debug Probe    |       | Microcontroller     |       | FPGA Target Board   |
| (Host Debug Master) |       | Target (STM32-class)|       | (Artix-7 class)     |
+-------+-------------+       +-------+-------------+       +-------+-------------+
        |                             |                             |
        | VTref, TCK, TMS, TDI, TDO   | JTAG / SWD Target Header    | JTAG Target Header
        +---------------------------->+ (TAP Node A)                | (TAP Node B)
        |                             +-----------------------------+
        +---------------------------------------------------------->+
```

## 2. Voltage Domain and Signal Interface Matrix

| Interface | Signal Group | Driver / Owner | Nominal Domain | Level Shifting | Connector Type | Engineering Assumptions |
|---|---|---|---|---|---|---|
| Host USB | VBUS, D+, D-, GND | Host Controller | 5.0 V / 3.3 V | Built-in PHY | USB Type-B / Type-C | Host supplies up to 500 mA per USB 2.0 port |
| MCU Power | VDD_MCU, GND | On-board LDO | 3.3 V | Not required | Terminal Block / 2.1mm DC | Derived from 5.0 V VBUS; common chassis GND |
| FPGA Core | VCCINT, GND | On-board Switcher | 1.0 V | Dedicated VRM | Integrated BGA planes | Isolated switcher rail; maximum ripple 30 mV |
| FPGA I/O | VCCO_0..3, GND | On-board Regulators | 1.8 V / 3.3 V | Bank selectable | Samtec / 0.1" Header | Bank 0 dedicated to configuration logic |
| JTAG TAP | TCK, TMS, TDI | Debug Adapter | VTref (1.2V - 3.3V) | Active bidirectional | 2x5 0.05" / 2x10 0.1" | VTref sensed by probe to bias level shifters |
| JTAG Return | TDO | Target TAP Controller | VTref (1.2V - 3.3V) | Active bidirectional | 2x5 0.05" / 2x10 0.1" | Driven synchronously on falling edge of TCK |
| JTAG Control | nSRST, nTRST | Debug Adapter | VTref (Open-Drain) | Pull-up to VTref | 2x5 0.05" / 2x10 0.1" | Optional reset pins with 10k pull-up |

## 3. Grounding and Signal Integrity
1. **Single-Point Grounding**: Common ground potential between Host, Debug Probe, MCU, and FPGA established through shielding and dedicated ground return paths.
2. **Target Sensing (VTref)**: The JTAG adapter monitors target VTref pin directly to set driver reference voltages and protect target I/O buffers from overvoltage stress.
3. **Safety Bring-Up Sequence**:
   - Power off all hardware before connecting JTAG debug lines.
   - Establish ground return prior to signal contact.
   - Verify power rail potentials prior to TAP chain scan.

