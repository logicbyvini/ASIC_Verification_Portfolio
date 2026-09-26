# Standard Verification IPs (VIPs)

This directory hosts reusable **Universal Verification Methodology (UVM 1.2)** compliant Verification IP (VIP) components for industry-standard bus protocols.

## Protocols Implemented
### 1. AMBA AXI4-Lite UVM Agent (`amba_axi_lite/`)
- **Features Supported:**
  - Configurable Active/Passive operation mode.
  - Parameterizable Address and Data bus widths.
  - Independent Read/Write channels according to the ARM AMBA AXI4-Lite Protocol Specification.
  - Built-in SystemVerilog Assertions (SVA) for protocol compliance checks (e.g., `VALID`/`READY` handshakes, stability during wait states).
  - Functional coverage collection for address range hit, back-to-back transfers, and response statuses (`OKAY`, `SLVERR`).
