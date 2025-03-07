#### CPU/GPU 구조![[Pasted image 20250226203439.png]]

#### 설계 역사
- 에너지 소모나 발열 이슈로 인하여 클럭 주파수와 한 클럭에서 수행할 수 있는 수행 능력을 증가시키는데 한계가 생김
- 이후 하나의 칩에 여러 개의 프로세서 유닛을 집적하여 연산 능력을 증가시키는 방향으로 전환됨

#### Multi-Core
- 순차 프로그램(Sequential Program)의 실행 속도 향상을 추구
	- 겉으로 보기에는 순차적으로 실행하지만, 실제로는 단일 스레드를 구성하는 명령어들은 병렬적으로(비순차적으로) 실행하는 복잡한 제어 로직을 사용함
	- 어플리케이션에서 명령어와 데이터 접근 지연시간을 줄이기 위해서 큰 캐시 메모리를 사용함
	- 그런데 복잡한 제어 로직이나 큰 캐시 메모리는 **연산 속도**에는 영향을 끼치지 못함
- Intel의 i9 프로세서의 경우에는 16개의 코어를 가지고 있고,
  각 코어는 2개의 하드웨어 스레드를 가지고 하이퍼스레딩(hyper-threading)을 지원하여 순차 프로그램의 실행 속도를 최대화하도록 설계됨

#### 메모리 대역폭
- 많은 어플리케이션의 속도는 메모리 시스템에서 프로세서로 전달해야 하는 메모리의 비율에 따라서 제한됨
- legacy OS, 어플리케이션, I/O 디바이스들의 요구사항을 만족해야 하기 때문에 메모리 대역폭을 늘리는 것은 어려움

#### 설계 철학
- Latency-Oriented Design
- 낮은 지연시간(low-latency)에서 단일 스레드 성능을 극대화