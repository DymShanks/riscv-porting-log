# RISC-V Porting Log
## Environment
- Host: x86_64 Ubuntu 24.04
- Toolchain: riscv64-linux-gnu-gcc 13.3.0, gfortran-riscv64-linux-gnu
- QEMU: 8.2.2 userspace

## CloverLeaf_Serial — HPC Hydrodynamics Mini-app
- Source: https://github.com/UK-MAC/CloverLeaf_Serial
- Category: HPC Benchmark (spreadsheet row 96)
- Status: ✅ COMPILED SUCCESSFULLY

### Fixes Required
1. MPI_COMPILER_GNU hardcoded as `gfortran` → changed to `riscv64-linux-gnu-gfortran`
2. `-march=native` passed to cross-compiler → removed (invalid for cross-compilation)
3. C_MPI_COMPILER_GNU hardcoded as `gcc` → changed to `riscv64-linux-gnu-gcc`

### Error Classification
- Class B: Missing cross-compiler (gfortran not installed for RISC-V)
- Class D: Makefile hardcodes native compiler names instead of using variables
- Class D: `-march=native` incompatible with cross-compilation

### Patch
Three `sed` one-liners fix all issues:
```bash
sed -i 's/MPI_COMPILER_GNU = gfortran/MPI_COMPILER_GNU = riscv64-linux-gnu-gfortran/' Makefile
sed -i 's/-march=native//' Makefile
sed -i 's/C_MPI_COMPILER_GNU = gcc/C_MPI_COMPILER_GNU = riscv64-linux-gnu-gcc/' Makefile
make COMPILER=GNU
```
