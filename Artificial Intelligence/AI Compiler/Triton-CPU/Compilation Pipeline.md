![[스크린샷 2025-05-24 오후 2.00.11.png]]

# Key Transformation Phases
### 1. TTIR → TTCIR
- Scalarization of operations
- Conversion of memory, pointer, and elementwise operations
- Transformation of dot products and reduction operations
### 2.  TTCIR → Target TTCIR:
- CPU-specific optimizations like mask optimization
- Conversion of dot operations to hardware-accelerated versions (AMX/FMA)
- Integration with micro-kernel libraries like OneDNN and XSMM
### 3. Final LLVM Lowering and Binary Generation:
- Lowering to LLVM IR
- LLVM optimization passes
- Code generation and linking into shared libraries

# Performance Considerations

- CPU 백엔드를 사용할 때, 아래의 요소들을 생각해서 작성해야 성능을 향상시킬 수 있음

#### 1. **Block Size**:
- The CPU backend maps Triton's blocks to CPU threads. Choosing appropriate block sizes is important for performance.

#### 2. **Memory Access Patterns**:
- CPU architectures are sensitive to memory access patterns. Linear access patterns are generally more efficient.

#### 3. **Vectorization**:
- The CPU backend attempts to use SIMD instructions. Operations on contiguous data typically vectorize better.

#### 4. **Hardware Features**:
- The backend can leverage specialized CPU features:
    - AMX (Advanced Matrix Extensions) on newer Intel CPUs
    - FMA (Fused Multiply-Add) instructions
- Optimized libraries (OneDNN, XSMM) for common operationsg