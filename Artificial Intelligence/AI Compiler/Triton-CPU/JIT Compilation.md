## Overview
![[스크린샷 2025-05-24 오후 2.19.28.png]]

## Compilation Process
![[스크린샷 2025-05-24 오후 2.20.31.png]]
## CPU-Specific Compilation Passes

The CPU backend applies several specialized compilation passes:
1. **Scalarization**: Converts block operations to scalar operations for CPU execution
2. **Memory Operation Conversion**: Transforms memory operations to match CPU memory model
3. **Dot Operation Conversion**: Specializes matrix multiplications for CPU execution
4. **Control Flow Conversion**: Adapts Triton control flow to CPU execution model
5. **Hardware-Specific Optimizations**:
    - AMX/FMA instruction generation on supported hardware
    - OneDNN/XSMM library integration for optimized kernels
## CPU Backend-Specific Features

The CPU backend offers specialized features:
1. **Threading Model**: Controls how many CPU threads to use for kernel execution via `num_threads`
2. **Vector Library Selection**: Choose between `libsleef` or `libmvec` for vectorized math functions
3. **Hardware-Specific Optimizations**:
    - AMX instruction support (automatically enabled when available)
    - AVX/AVX-512 instruction generation
    - FMA instruction optimization
4. **Microkernel Libraries**:
    - OneDNN integration for optimized kernels
    - XSMM support for Small Matrix Multiplications