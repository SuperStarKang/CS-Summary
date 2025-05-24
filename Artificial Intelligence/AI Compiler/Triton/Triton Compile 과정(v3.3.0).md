1. **커널 함수 정의**: 사용자가 `@triton.jit` 데코레이터를 사용하여 Triton 커널 함수를 정의
2. **커널 함수 호출**: 사용자가 정의한 커널 함수를 호출하면, `JITFunction.run` 메서드를 실행
	- python/triton/runtime/jit.py:525-526
3. **컴파일 과정**: 이 과정에서 `compiler.py`의 `compile` 함수가 호출됨
	- Backend 정보를 가져와서 CUDA인지, AMD인지 파악하여 설정
		- 이 정보를 이용해서 어떤 기기의 언어로 생성할지 미리 파악함
	- python/triton/runtime/jit.py:568-570
4. **컴파일 파이프라인**: `compile` 함수는 Python AST에서 LLVM IR까지의 변환 과정을 관리함
	- python/triton/compile/compiler.py:214
5. **단계별 변환**: 컴파일 과정에서 각 단계(TTIR → TTGIR → LLVM IR)별 변환이 이루어짐
	- python/triton/compile/compiler.py:283-284
	- nvidia backend: python/triton/backends/nvidia/compiler.py 사용
		- nvidia backend에 맞게 make ttir,ttgit,llir,ptx,cubin 함수들이 구현되어 있음
	- TTIR, TTGIR, PTX, CUBIN 각각의 파일들을 생성
	- 결과로 CompileKernel 반환
6. 실행: `JITFunction.run` 메서드에서 컴파일된 커널을 실행함
	- python/triton/runtime/jit.py:590-591

## C++ 등록 구조
- Triton의 C++ 함수들은 Python에 등록되어 있어 Python 코드에서 호출할 수 있음
1. C++ 함수 등록: C++ 함수들은 pybind11을 사용하여 Python에 등록
	- python/src/passes.cc & python/src/passess.h
2. Python에서 C++ 호출: Python 코드에서는 등록된 C++ 함수를 직접 호출할 수 있음
	- python/triton/compile/compiler.py:276-277
3. **CUDA 유틸리티 함수 등록**: CUDA 관련 C++ 함수들도 Python에 등록되어 있음
	- python/triton/backends/nvidia/driver.py
4. **런처 생성**: 커널 실행을 위한 런처도 C++로 구현되어 Python에 등록되어 있음
	- python/triton/backends/nvidia/drivec.c