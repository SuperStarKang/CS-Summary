## Autotune?
- 자동으로 가장 최적화된 파라미터들을 찾는 것

## Autotune 사용
- `@triton.autotune` 데코레이터를 통해 커널 함수에 적용
- pre-hook을 포함한 설정도 가능
	```python
@triton.autotune(
	configs=matmul_get_configs(pre_hook=matmul_tma_set_block_size_hook),
	key=["M", "N", "K", "WARP_SPECIALIZE"],
)
```
## Autotune 실행 과정
- python/triton/runtime/autotuner.py에 구현되어 있음
	1. 사용자가 `@triton.autotune` 데코레이터를 사용하여 커널 함수를 정의
	2. 커널 함수가 처음 호출될 때, Autotune은 제공된 모든 설정으로 커널을 실행하고 성능을 측정
	3. 가장 빠른 설정을 찾아 캐시하고, 이후 호출에서는 이 설정을 사용