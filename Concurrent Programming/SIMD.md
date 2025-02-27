- Single Instruction Multiple Data
- CPU에서 지원되는 명령어 셋으로 하나의 명령어로 동일한 형태/구조의 여러 데이터를 한 번에 처리하는 병렬처리기법
#### SIMD 연산
![[Pasted image 20250227233626.png]]
- 4개의 32비트 정수 A0, A1, A2, A3와 4개의 32비트 정수 B0, B1, B2, B3를 각각 덧셈 연산할 때
- SIMD 연산에서는 1번의 연산으로 처리가 가능함

#### 코드
- C
	![[Pasted image 20250227233849.png]]
- 어셈블리 변환
	![[Pasted image 20250227233931.png]]
- 1회의 paddd instruction을 포함하여 총 6개의 instruction 밖에 사용하지 않음