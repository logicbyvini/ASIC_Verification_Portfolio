# Advanced ASIC Verification Portfolio

Welcome to my Hardware Verification Portfolio. This repository serves as a centralized hub for my projects in **Digital ASIC Verification**, focusing on modern industry methodologies, standard protocols, and complex data path architectures.

**Author:** Vinícius Eduardo Lopes da Silva  
**Role:** Junior ASIC Verification Engineer  
**Contact:** [LinkedIn](https://linkedin.com/in/vinícius-lopes-282937151) | viniedulop43@gmail.com

---

## 🛠️ Technical Stack & Methodologies
- **Verification:** UVM, SystemVerilog, Assertions (SVA), Formal Verification.
- **Architectures:** Clock Domain Crossing (CDC), Async FIFOs, High-Speed Interconnects, AI Accelerators.
- **Protocols:** AMBA AXI4-Lite, APB, PCIe/UCIe Concepts, UART, SPI, I2C.
- **EDA Tools:** Synopsys (VCS, Verdi, SpyGlass CDC), Cadence (Xcelium, SimVision).
- **Automation:** Python, Tcl, Bash, Makefiles.

---

## 📂 Repository Structure

```text
Digital_Verification_Portfolio/
│
├── 01_CNN_Accelerator_DV/                  # Verification Environment for a 1D-CNN ASIC
│   ├── docs/                               # Test Plans, Verification Strategy, and Bug Tracking
│   ├── uvm_tb/                             # UVM Testbench (Env, Agents, Scoreboards, Sequences)
│   └── README.md                           # Details about the CNN Verification approach
│
├── 02_High_Speed_Interconnect_Concepts/    # Concepts for D2D/Chiplet communications
│   ├── async_fifo/                         # Gray-coded Async FIFO design and verification
│   ├── cdc_analysis/                       # Clock Domain Crossing test cases and analysis
│   └── README.md                           
│
├── 03_Standard_VIPs/                       # Verification IPs developed for standard protocols
│   └── amba_axi_lite/                      # UVM Agent for AXI4-Lite protocol
│
├── scripts/                                # Automation scripts for EDA tools
│   ├── Makefile                            # Unified Makefile for CLI debugging and regressions
│   └── run_sim.sh                          # Bash wrappers for simulation environments
│
├── .gitignore                              # Rules to ignore EDA generated files (csrc, simv, fsdb)
└── README.md                               # This file
