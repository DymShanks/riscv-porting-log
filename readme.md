# RISC-V HPC Porting Log & Automation Pipeline

**LFX Mentorship Summer 2026** — *Broadening the RISC-V High Precision Code Base* **Applicant:** Divyam Shankhdhar ([@DymShanks](https://github.com/DymShanks))  
**Mentor:** Kurt Keville (MIT)  

---

## 🎯 Repository Purpose

This repository serves as the central working directory and proof-of-work for my LFX Summer 2026 mentorship proposal. It contains the initial automation tooling, empirical error classification data, and verified porting logs for the first batch of HPC scientific codes ported from `x86_64` to `riscv64`.

Rather than treating the 400-code spreadsheet as a manual task, this project takes an **automation-first approach** to systematically cross-compile the ecosystem.

## 🧰 Core Tooling: `risc-v-porter.py`

The crown jewel of this repository is the `risc-v-porter.py` script. It implements a 6-stage CI/CD-style pipeline designed to convert the porting effort from $O(n)$ manual work to $O(1)$ human attention per error class.

**The 6-Stage Pipeline:**
1. **Detect:** Identifies the build system (`cmake`, `make`, `autotools`).
2. **Attempt:** Initiates cross-compilation with the `riscv64-linux-gnu` toolchain.
3. **Classify:** Pattern-matches `stderr` against an empirical 5-class taxonomy (Classes A through E).
4. **Auto-Fix:** Applies targeted AST-level/sed patches (e.g., guarding `xmmintrin.h`, stripping `-march=native`).
5. **Retry:** Re-attempts the build automatically.
6. **Verify:** Runs the resulting binary under `qemu-riscv64` to confirm RISC-V ELF execution.

## ✅ Pre-Proposal Proof of Work

To validate the automation pipeline before submitting the proposal, two target codes from the mentorship spreadsheet were successfully ported and verified running on RISC-V emulators.

| Code | Spreadsheet Row | Error Class | Applied Fixes | Status |
| :--- | :--- | :--- | :--- | :--- |
| **CloverLeaf** | 96 | `Class D` (Build System) | Hardcoded compilers replaced; `-march=native` stripped. | 🟢 **Running (ELF 64-bit UCB RISC-V)** |
| **RAxML** | 97 | `Class C` (x86 Intrinsics) | `-msse3` stripped; `xmmintrin.h` guarded; `_mm_setcsr` stubbed. | 🟢 **Running (ELF 64-bit UCB RISC-V)** |

*Note: Graph500 (Row 95) was correctly classified as `Class B` (Missing Dep: mpicc) by the pipeline. It will be unblocked in Phase 2 once the OpenMPI sysroot is cross-compiled.*

## 🚀 Usage

To test the pipeline locally using the dry-run flag (classification only, no source modifications):

```bash
# Attempt a single code
python3 risc-v-porter.py --attempt ./RAxML --name RAxML --dry-run

# Run the batch classifier
python3 risc-v-porter.py --batch codes.json
