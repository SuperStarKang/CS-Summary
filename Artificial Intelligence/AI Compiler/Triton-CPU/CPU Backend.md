## 0. Triton IR Generation
```python
def make_ttir(mod, metadata, opt):
	# This is the same as the Nvidia backend.
	pm = ir.pass_manager(mod.context)
	pm.enable_debug()
	passes.common.add_inliner(pm)
	passes.ttir.add_combine(pm)
	passes.common.add_canonicalizer(pm)
	passes.ttir.add_reorder_broadcast(pm)
	passes.common.add_cse(pm)
	passes.common.add_licm(pm)
	passes.common.add_symbol_dce(pm)
	pm.run(mod)
	
	return mod
```
- Inlining
- Operation combining
- Canonicalization
- Broadcast reordering
- Common subexpression elimination
- Loop-invariant code motion
- Dead code elimination
## 1. **TTIR to TTCIR Conversion**:
![[Pasted image 20250524165033.png]]
- **Scalarize**
	- TTIR은 벡터 기반 연산(vectorized operation)이 중심인데, CPU는 보통 **스칼라 연산**(단일 값 처리)에 최적화됨
	- 따라서 TTIR에서 벡터 연산을 **loop와 scalar op**로 분해
- **Convert Memory Operations**
	- `tt.load`, `tt.store` 같은 추상적인 메모리 연산을 CPU에 맞는 **명시적 메모리 접근**으로 변환
- **Convert Pointer**
	- TTIR의 포인터 계산 (`make_ptr`, `advance`) 등을 C 스타일 포인터 연산으로 바꾸는 과정
- **Convert Elementwise Ops**
	- GPU용 `tt.add`, `tt.mul` 등의 벡터화된 연산을 CPU용 스칼라/루프 기반 연산으로 바꿈
- **Convert Dot Op**
	- `dot(a, b)` 같은 연산을 일반적인 행렬 곱으로 변환
	- BLAS 호출 또는 직접 루프 기반 구현으로 lowering
- **Convert Reduction Op**
	- `sum`, `max`, `min` 같은 reduction 연산을 loop 기반으로 구현
- **Convert Control Flow Ops**
	- GPU에서는 warp/thread 단위로 branch를 수행하지만, CPU는 일반적인 조건문(if, for)을 사용하므로 변환이 필요
- **Convert Atomic Ops**
	- GPU의 `atomic_add`, `atomic_max` 같은 연산은 CPU에서 `std::atomic` 또는 `mutex` 등을 통해 구현되어야 함
## 2. **TTCIR to Target TTCIR**:
![[Pasted image 20250524165701.png]]
- 아키텍처 특성을 고려한 **하드웨어 최적화** 변환
	- 결과물로 생성된 IR은 **LLVM IR로 변환할 준비가 완료된 형태**
	- CPU에 **최적화된 연산 패턴**과 **타겟 ISA에 맞는 형태로 lowering**
- **CPU Canonicalizer**
	- **목적**: TTCIR의 기본 연산을 CPU에서 쉽게 다룰 수 있는 정규 형태로 변환
	- **필요성**: 이후 최적화 패스(AMX, FMA 등)가 예상하는 형태로 코드를 만들어주는 준비 과정
- **Optimize Masks**
	- **목적**: 조건부 연산(마스크 연산) 최적화
	- **필요성**: CPU에서는 마스크 연산이 성능 저하 요인이 되므로, 이를 효율적으로 처리할 수 있는 형태로 최적화
- **Dot Operation Optimizations**
	- **중첩된 분기 구조**로 dot 연산 최적화 방식 선택
	- 아키텍처에 맞게 변환
	1. **OneDNN / XSMM Conversion**
		- **조건**: AVX512, AVX2 등이 탑재된 x86_64 CPU에서 사용
		- **역할**: Triton의 matmul 연산을 Intel의 highly optimized library (OneDNN or libxsmm) 호출로 변환
		- **필요성**: 수작업으로 loop 짜는 것보다 훨씬 고성능
	2. **AMX Conversion**
		- **조건**: `amx-int8`, `amx-bf16` 플래그가 있는 경우
		- **역할**: AMX (Advanced Matrix Extension)를 이용한 고속 행렬 곱 연산으로 변환
		- **필요성**: Intel Sapphire Rapids 같은 최신 CPU에서 **행렬 연산을 가속**하는 데 필수
	3. **FMA Conversion**
		- **조건**: AVX512F 또는 AVX2 같은 Fused Multiply-Add 지원 CPU
		- **역할**: `a*b + c` 형태를 **FMA 명령어**로 lowering
		- **필요성**: 성능 및 정밀도 개선
	4. **Generic Dot Conversion**
		- **조건**: 특수 하드웨어 없음 (fallback)
		- **역할**: CPU에서 수동 loop 기반으로 dot 연산 구현
		- **필요성**: 호환성 확보 (모든 CPU에서 fallback 가능)
- **Convert Unsupported Ops**
	- **역할**: 해당 하드웨어에서 지원되지 않는 연산들을 우회하거나 대체
	- **예시**:
	    - FP8 연산을 FP32로 프로모션
	    - 혼합정밀도 연산을 분리된 연산으로 치환
		- **필요성**: 정밀도 손실 방지, 비호환 연산 우회 처리
- **Decompose FP Conversions**
	- **역할**: 복잡한 float 포맷 (예: FP16, BF16, FP8) 연산을 FP32 기반으로 재작성
	- **조건**:
	    - `avx512bf16` 없는 경우 → BF16 → FP32
	    - FP8은 무조건 FP32로 전환
	- **필요성**: 현재 대부분의 CPU가 FP8/BF16 네이티브 연산 미지원이므로, 정확도를 위한 promotion 필요
## 3. **Target TTCIR to LLVM IR**:
![[스크린샷 2025-05-24 오후 5.10.28.png]]
- **[[Ukernel]]s to LLVM Runtime Calls**
	- **역할**: Triton 내부 연산을 성능 좋은 **라이브러리 호출**로 대체
	- **예시**:
	    - `dot(a, b)` → `libxsmm_gemm(a, b)`
	    - `matmul(a, b)` → `onednn_matmul(a, b)`
	- **필요성**: Triton이 직접 loop unroll해서 matmul을 돌리는 것보다, 잘 최적화된 라이브러리를 호출하는 것이 훨씬 빠름
- **Vector Operation Lowering**
	- **역할**: 고차원 벡터 연산을 lower dimension으로 분해하거나, 실제 SIMD 명령어 형태로 변환
	- **예시**: `%vec = vector<8xf32> add` → `<llvm.simd.add intrinsic>`
- **Memory Operation Conversion**
	- **역할**: `memref.load/store` 등 메모리 연산을 LLVM의 `load`, `store`로 변환
- **Math Function Conversion**
	- **역할**: `exp`, `sqrt`, `log` 등 수학 함수를 아래의 우선순위로 변환
		1. 벡터화 수학 라이브러리 (예: libsleef, libmvec)
		2. 기본 C 수학 라이브러리 (libm)
		3. LLVM 내장 연산
	- **선택 기준**: `VecLib.libsleef`, `libmvec` 등은 AVX, NEON 등 CPU 특징에 따라 선택
- **MLIR Dialect Conversions**
	- **역할**: 남은 MLIR dialect (e.g., `scf`, `affine`, `index`, `arith`)를 LLVM IR로 lowering
- **Final LLVM IR Passes**
	- **역할**: 최종 함수/변수/디버깅 정보 포함한 LLVM IR 완성
- Optimization (LLVM 레벨)
	- **역할**: `-O3` 수준으로 최적화
## 4. **LLVM IR to Executable**:
    
    - Translates to platform-specific assembly
    - Compiles to shared object for dynamic loading