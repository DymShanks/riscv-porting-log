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

## RAxML — Phylogenetics (row 97)
- Source: https://github.com/stamatak/standard-RAxML
- Status: ✅ COMPILED AND RUNNING

### Fixes Required
1. Used Makefile.gcc instead of Makefile.SSE3.gcc
2. Removed -msse flags: `sed -i 's/-msse[^ ]*//g' Makefile.gcc`
3. Guarded xmmintrin.h: `#ifndef __x86_64__`
4. Stubbed _mm_setcsr call at line 13723

### Error Classification
- Class C: x86 SSE flags (-msse, -msse3) — invalid on RISC-V
- Class C: x86 SSE headers (xmmintrin.h)
- Class C: x86 SSE runtime intrinsics (_mm_setcsr)

### Automation Pattern
All Class C fixes are scriptable:
  sed -i 's/-msse[^ ]*//g' Makefile
  sed -i 's/#include <xmmintrin.h>/#ifndef __x86_64__\n\/\/skipped\n#endif/' *.c
