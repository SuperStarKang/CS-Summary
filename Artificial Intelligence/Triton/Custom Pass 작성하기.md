## Custom Pass 적용 과정

1. Custom C++ Pass 작성
2. MLIR Pass registry를 통해 Pass 등록
3. Transformation Logic 구현
4. Triton 컴파일 파이프라인에 Pass 추가

## TTIR의 모든 ADD 연산을 MUL로 바꿔보기
### Step 1: Create a Pass File
- `lib/Dialect/Triton/Transforms/` 디렉토리에 `AddToMulConversion.cpp` 작성
### Step 2: Register the Pass
- `include/triton/Dialect/Triton/Transforms/Passes.h`헤더 파일에 Pass 등록
### Step 3: Add the Pass to the Compilation Pipeline
### Step 4: Expose the Pass to Python
### Step 5: Use the Pass in Python