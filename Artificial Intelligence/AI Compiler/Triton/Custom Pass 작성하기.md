## Custom Pass 적용 과정

1. Custom C++ Pass 작성
2. MLIR Pass registry를 통해 Pass 등록
3. Transformation Logic 구현
4. Triton 컴파일 파이프라인에 Pass 추가

## TTIR의 모든 ADD 연산을 MUL로 바꿔보기
### Step 1: Create a Pass File
- `lib/Dialect/Triton/Transforms/` 디렉토리에 `AddToMulConversion.cpp` 작성
```c++
#include <memory>

#include "mlir/Dialect/SCF/Utils/Utils.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Support/LogicalResult.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "triton/Analysis/Utility.h"
#include "triton/Dialect/Triton/IR/Dialect.h"
#include "triton/Dialect/Triton/Transforms/Passes.h"
#include "llvm/Support/Debug.h"

#define GEN_PASS_CLASSES
#include "triton/Dialect/Triton/Transforms/Passes.h.inc"

using namespace mlir;

namespace mlir::triton {

namespace {
	// Pattern to replace add with mul
    struct AddToMulPattern : public OpRewritePattern<arith::AddFOp> {
        using OpRewritePattern<arith::AddFOp>::OpRewritePattern;
        LogicalResult matchAndRewrite(arith::AddFOp op,
                                    PatternRewriter &rewriter) const override {
            // Create a multiplication operation to replace the addition
            rewriter.replaceOpWithNewOp<arith::MulFOp>(op, op.getType(),
                                                    op.getLhs(), op.getRhs());
            return success();
        }
    };

    // Pattern for integer addition
    struct AddIToMulPattern : public OpRewritePattern<arith::AddIOp> {
        using OpRewritePattern<arith::AddIOp>::OpRewritePattern;
        LogicalResult matchAndRewrite(arith::AddIOp op,
                                    PatternRewriter &rewriter) const override {
            // Create a multiplication operation to replace the addition
            rewriter.replaceOpWithNewOp<arith::MulIOp>(op, op.getType(),
                                                    op.getLhs(), op.getRhs());
            return success();
        }
    };

    // The actual pass
    struct AddToMulPass : public PassWrapper<AddToMulPass, OperationPass<ModuleOp>> {
        MLIR_DEFINE_EXPLICIT_INTERNAL_INLINE_TYPE_ID(AddToMulPass)

		void getDependentDialects(DialectRegistry &registry) const override {
            registry.insert<arith::ArithDialect, triton::TritonDialect>();
        }

        StringRef getArgument() const final {
            return "convert-add-to-mul";
        }

        StringRef getDescription() const final {
            return "Convert all addition operations to multiplication operations";
        }

        void runOnOperation() override {
            MLIRContext *context = &getContext();
            ModuleOp module = getOperation();

            // Set up patterns
            RewritePatternSet patterns(context);
            patterns.add<AddToMulPattern, AddIToMulPattern>(context);

            // Apply the patterns
            if (failed(applyPatternsGreedily(module, std::move(patterns)))) {
                signalPassFailure();
            }
        }
    };
} // end anonymous namespace

  

// Pass registration
std::unique_ptr<Pass> createAddToMulPass() {
    return std::make_unique<AddToMulPass>();
}

} // namespace mlir::triton
```
### Step 2: Register the Pass
- `include/triton/Dialect/Triton/Transforms/Passes.h`헤더 파일에 Pass 등록
```c++
#ifndef TRITON_DIALECT_TRITON_TRANSFORMS_PASSES_H_
#define TRITON_DIALECT_TRITON_TRANSFORMS_PASSES_H_

#include "mlir/Pass/Pass.h"

namespace mlir {

    namespace triton {
        std::unique_ptr<Pass> createCombineOpsPass();

        std::unique_ptr<Pass> createReorderBroadcastPass();
        std::unique_ptr<Pass> createRewriteTensorPointerPass();
        std::unique_ptr<Pass> createLoopUnrollPass();

        // custom pass
        std::unique_ptr<Pass> createAddToMulPass();
    } // namespace triton

    #define GEN_PASS_REGISTRATION
    #include "triton/Dialect/Triton/Transforms/Passes.h.inc"

} // namespace mlir

#endif
```
- `include/triton/Dialect/Triton/Transforms/Passes.td` 테이블 파일에 Pass 등록
```
def ConvertAddToMul : Pass<"convert-add-to-mul", "mlir::ModuleOp"> {  
  let summary = "Convert all addition operations to multiplication operations";  
  let description = [{  
    This pass replaces all addition operations with multiplication operations.  
    It's a demonstration of how to create custom transformation passes in Triton.  
  }];  
  let constructor = "mlir::triton::createAddToMulPass()";  
  let dependentDialects = ["mlir::arith::ArithDialect", "mlir::triton::TritonDialect"];  
}
```
- triton/lib/Dialect/Triton/Transforms/CMakeLists.txt 빌드 리스트에 Custom 파일 추가
```txt
set(LLVM_TARGET_DEFINITIONS Combine.td)
mlir_tablegen(TritonCombine.inc -gen-rewriters)
add_public_tablegen_target(TritonCombineIncGen)

add_triton_library(TritonTransforms
  Combine.cpp
  LoopUnroll.cpp
  ReorderBroadcast.cpp
  RewriteTensorPointer.cpp

  AddToMulConversion.cpp

  DEPENDS
  TritonTransformsIncGen
  TritonCombineIncGen

  LINK_LIBS PUBLIC
  MLIRPass
  MLIRTransformUtils
  TritonIR
)
```
### Step 3: Add the Pass to the Compilation Pipeline
- python/src/passes.cc 파일에 해당하는 단계에 Pass 추가
```c++
void init_triton_passes_ttir(py::module &&m) {
  using namespace mlir::triton;
  ADD_PASS_WRAPPER_0("add_combine", createCombineOpsPass);
  ADD_PASS_WRAPPER_0("add_reorder_broadcast", createReorderBroadcastPass);
  ADD_PASS_WRAPPER_0("add_rewrite_tensor_pointer",
                     createRewriteTensorPointerPass);
  ADD_PASS_WRAPPER_0("add_loop_unroll", createLoopUnrollPass);
  ADD_PASS_WRAPPER_0("add_convert_add_to_mul", createAddToMulPass);
  ADD_PASS_WRAPPER_4("add_convert_to_ttgpuir",
                     createConvertTritonToTritonGPUPass, const std::string &,
                     int, int, int);
}
```
- 작업 환경에 맞는 GPU에 추가해야 함
- NVIDIA의 경우 third_party/nvidia/backend/compiler.py 파일에 해당하는 단계에 Pass 추가
```python
    @staticmethod
    def make_ttir(mod, metadata, opt):
        pm = ir.pass_manager(mod.context)
        pm.enable_debug()
        passes.common.add_inliner(pm)
        passes.ttir.add_rewrite_tensor_pointer(pm)
        passes.common.add_canonicalizer(pm)
        passes.ttir.add_combine(pm)
        passes.ttir.add_reorder_broadcast(pm)
        passes.common.add_cse(pm)
        passes.common.add_symbol_dce(pm)
        passes.ttir.add_loop_unroll(pm)

        # Custom Pass
        passes.ttir.add_convert_add_to_mul(pm)

        pm.run(mod)
        return mod
```
### Step 4: Build
- triton 최상위 디렉토리에서 pip install -e python 실행
- 이 과정이 3시간 이상 걸리기 때문에, 수정한 부분만 빌드하는 방법을 찾아야함..
### Step 5: Test
- 테스트 함수
```python
@triton.jit

def add_kernel(x_ptr, y_ptr, output_ptr, n_elements, BLOCK_SIZE: tl.constexpr):
    pid = tl.program_id(0)
    offsets = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = offsets < n_elements
    x = tl.load(x_ptr + offsets, mask=mask)
    y = tl.load(y_ptr + offsets, mask=mask)

    y = x + x
    out = x + y

    tl.store(output_ptr + offsets, out, mask=mask)
```
- 결과
```ttir
module {
  tt.func public @add_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32} loc("/home/kangmlee/Triton/workspace/Research/test.py":6:0), %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32} loc("/home/kangmlee/Triton/workspace/Research/test.py":6:0), %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32} loc("/home/kangmlee/Triton/workspace/Research/test.py":6:0), %arg3: i32 {tt.divisibility = 16 : i32} loc("/home/kangmlee/Triton/workspace/Research/test.py":6:0)) attributes {noinline = false} {
    %c128_i32 = arith.constant 128 : i32 loc(#loc1)
    %0 = tt.get_program_id x : i32 loc(#loc2)
    %1 = arith.muli %0, %c128_i32 : i32 loc(#loc3)
    %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32> loc(#loc4)
    %3 = tt.splat %1 : i32 -> tensor<128xi32> loc(#loc5)
    %4 = arith.muli %3, %2 : tensor<128xi32> loc(#loc5)
    %5 = tt.splat %arg3 : i32 -> tensor<128xi32> loc(#loc6)
    %6 = arith.cmpi slt, %4, %5 : tensor<128xi32> loc(#loc6)
    %7 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>> loc(#loc7)
    %8 = tt.addptr %7, %4 : tensor<128x!tt.ptr<f32>>, tensor<128xi32> loc(#loc7)
    %9 = tt.load %8, %6 : tensor<128x!tt.ptr<f32>> loc(#loc8)
    %10 = arith.mulf %9, %9 : tensor<128xf32> loc(#loc9)
    %11 = arith.mulf %9, %10 : tensor<128xf32> loc(#loc10)
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>> loc(#loc11)
    %13 = tt.addptr %12, %4 : tensor<128x!tt.ptr<f32>>, tensor<128xi32> loc(#loc11)
    tt.store %13, %11, %6 : tensor<128x!tt.ptr<f32>> loc(#loc12)
    tt.return loc(#loc13)
  } loc(#loc)
} loc(#loc)
```