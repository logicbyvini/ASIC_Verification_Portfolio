# ASIC Digital Verification Portfolio

Welcome to my Digital Verification portfolio. This repository serves as a centralized hub for my verification environments, test plans, and architectural studies targeting complex digital ASICs.

**Author:** Vinícius Eduardo Lopes da Silva  
**Role:** Digital Design & Verification Engineer  
**Contact:** [LinkedIn](https://linkedin.com/in/vinícius-lopes-282937151) | viniedulop43@gmail.com

---

## ⚠️ Important Disclaimer regarding IP Protection
> **Note:** For academic and proprietary projects (such as the CNN Hardware Accelerator), the **Synthesizable RTL (Design) files have been intentionally omitted** to protect Intellectual Property (IP). These directories strictly showcase the **Verification Infrastructure** (UVM components, Test Plans, Reference Models, and Scripting).

---

## 📂 Repository Structure & Projects

### [1. High-Speed Interconnect & CDC Concepts](./01_High_Speed_Interconnect_Concepts)
Focused on the critical challenges of Die-to-Die (D2D) and high-speed communications (e.g., UCIe, PCIe).
* **Async FIFO (CDC):** Implementation and verification of an Asynchronous FIFO using Gray code pointers to mitigate metastability across different clock domains.
* **SerDes Fundamentals:** Transaction-level verification of Serializer/Deserializer datapaths.

### [2. CNN Accelerator UVM Environment (Capstone Project)](./02_CNN_Accelerator_DV_Env)
Verification environment for a 1D-CNN Hardware Accelerator targeting Ventricular Fibrillation detection.
* **UVM Testbench:** Architecture of Agents, Virtual Sequences, and Scoreboards.
* **Documentation:** Detailed Verification Test Plan and Root Cause Analysis (RCA) logs.
* **Reference Model:** Bit-accurate C++/Python models used for golden reference checking.

### [3. AMBA Protocols Integration](./03_AMBA_Protocols)
* Verification IP (VIP) integration for **AXI4-Lite** utilized in RISC-V SoC environments.

### [4. Automation & EDA Scripts](./scripts_and_automation)
* Standardized `Makefiles` for regression runs and CLI debugging.
* Tcl scripts for environment setup across multi-vendor tools (Cadence/Synopsys).

---

## 🛠️ Technical Stack & Methodologies
* **Languages:** SystemVerilog (IEEE 1800), Python, C/C++, Bash, Tcl.
* **Methodologies:** UVM, Assertion-Based Verification (SVA), Constrained Random Testing (CRT), Coverage-Driven Verification (CDV).
* **EDA Tools (Experience):** 
  * *Cadence:* Xcelium, SimVision, vManager.
  * *Synopsys:* VCS, Verdi, SpyGlass CDC.

📁 ASIC_Verification_Portfolio/
│
├── 📁 01_High_Speed_Interconnect_Concepts/  # Seu foco para essa semana (HCLTech)
│   ├── 📁 async_fifo_cdc/                   # O projeto da FIFO Assíncrona
│   └── 📁 serdes_fundamentals/              # Lógica de serialização
│
├── 📁 02_CNN_Accelerator_DV_Env/            # O seu TCC (Sem o RTL)
│   ├── 📁 docs/                             # Test Plan e Root Cause Analysis
│   ├── 📁 reference_model/                  # C++ / Python model
│   └── 📁 uvm_tb/                           # Agentes, Sequencers, Scoreboard
│
├── 📁 03_AMBA_Protocols/                    # Para reaproveitar o que você fez no CI-Digital
│   └── 📁 axi4_lite_vip/
│
├── 📁 scripts_and_automation/               # Recrutadores de DV amam quem sabe automatizar
│   ├── makefile_templates/
│   └── tcl_eda_setup/
│
├── .gitignore
└── README.md

---
*Dedicated to building robust, coverage-driven, and scalable hardware.*
