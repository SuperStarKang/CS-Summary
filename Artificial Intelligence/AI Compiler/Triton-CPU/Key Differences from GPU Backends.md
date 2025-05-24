## 1. Execution Model:
- CPU: Thread-based parallelism with standard thread pools
- GPU: SIMT model with warps and blocks
## 2. **Memory Model**:
- CPU: Unified memory with cache hierarchy
- GPU: Complex memory hierarchy with shared memory, global memory, etc.
## 3. **Optimization Focus**:
- CPU: Vectorization, cache optimization, library integration
- GPU: Memory coalescing, shared memory usage, occupancy
## 4. **Hardware Acceleration**:
- CPU: AMX, AVX-512, NEON vector instructions
- GPU: Tensor cores, special hardware units