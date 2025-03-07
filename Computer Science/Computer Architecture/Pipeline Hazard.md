#### 구조적 해저드(Structural Hazard)
- 같은 클럭 사이클에 실행하기를 원하는 명령어의 조합을 하드웨어가 지원할 수 없기 때문에 발생
	- 주어진 클럭 사이클에 실행되도록 되어 있는 명령어 조합을 하드웨어가 지원하지 못해서 계획된 명령어 조합을 하드웨어가 지원하지 못해서 계획된 명령어가 적절한 클럭 사이클에 실행될 수 없는 사건
- 실생활 예시
	- 세탁기와 건조기가 같이 붙어 있는 기계를 사용하는 경우
	- 친구가 다른 일을 하느라고 바빠서 빨래를 옷장에 넣지 못하는 경우
- 프로그램 예시
	- 명령어와 데이터가 ==하나의 메모리==에 존재한다고 했을 때,
	  IF와 MEM 단계가 하나의 사이클에 있을때 실행 불가능
- 해결책
	- ==명령어와 데이터가 각각의 메모리==를 가지고 있으면 해결됨
	- 또는 명령어와 데이터가 각각의 캐시를 가지고 있으면 해결됨
- 파악이 쉽고, 설계 단계에서 미리 처리해주면 됨
=> 사실 큰 문제 X

#### 데이터 해저드(Data Hazard)
- 어떤 단계가 다른 단계가 끝나기를 기다려야 하기 때문에 파이프라인이 지연되어야 하는 경우
	- 명령어를 실행하는데 ==필요한 데이터가 아직 준비되지 않아서== 계획된 명령어가 적절한 클럭 사이클에 실행될 수 없는 사건
- 어떤 명령어가 아직 파이프라인에 있는 앞선 명령어에 종속성을 가질 때 데이터 해저드가 일어남
- 실행활 예시
	- 옷을 개다가 한 짝이 없는 양말을 발견했을 경우
	  => 다른 한 짝을 찾는 동안에 건조 과정이 끝난 후 개는 과정을 기다리는 옷들과 세탁 과정을 끝내고 건조 과정을 기다리는 옷들은 기다려야만 함
- 프로그램 예시
	- add 명령어 바로 다음에 add의 합을 상요하는 뺄셈 명령어가 뒤따르는 경우
	1. 
        `add $s0, $t0, $t1`
		`sub $t2, $s0, $t3
		`=> $s0 데이터에 대한 종속성 발생
	2. 
		`sum = a + b
		`avg = sum / 2
		`=> sum에 대한 종속성 발생
- 해결책
	- ==전방전달(Forwarding aka Bypassing)==

#### 제어 해저드(Control Hazard)
- 다른 명령어들이 실행되는 동안 어떤 명령어의 결과에 기반을 둔 결정을 할 필요가 있을 때 발생
- 분기 명령의 결과가 흐름을 결정
	- 분기 해저드(Branch Hazard)라고도 부름
- 실행활 예시
	- 축구 유니폼을 세탁하는 임무가 주어졌을 때, 세탁물이 더러운 정도에 맞추어 세제 농도와 물 온도를 결정해서 ==때는 잘 빠지지만 옷감이 상할 정도는 아니게 해야함==
	- 세탁소 파이프라인에서는 ==세탁기 설정을 바꿀 필요가 있는지 없는지를 결정하려면 두 번째 단계까지 기다려서== 마른 유니폼을 조사해야함
- 해결책
	1. 분기 시 ==지연==(Stall on Branch): 정확성을 위해
		- ID 단계에서 하드웨어적으로 처리
			![[Untitled (1).jpeg]]
		- 성능
			![[Untitled (2).jpeg]]
	2. ==예측(Prediction)==
		- 파이프라인이 긴 경우에 분기를 두 번째 단계(ID)에서 다 해결하지 못한다면 분기 명령어마다 지연시키는 것은 훨씬 더 큰 속도 저하를 초래할 것임
		  -> 분기에 대해(분기가 될 것인지, 안 될 것인지) ==예측==하여 진행
		1. 분기가 되지 않는다고 가정하고 진행
			![[Untitled (3).jpeg]]
			![[Untitled (4).jpeg]]
		2. 좀 더 정교한 분기 예측(branch prediction)
			- 어떤 경우는 분기한다고(taken) 예측하고,
			  어떤 경우는 분기하지 않는다고(untaken) 예측
				- ex. for문의 경우 현재 위치보다 작은 주소로 점프하는 분기가 항상 일어난다고 예측
		- 이러한 분기 예측 방법들은 보편적 행동에 의존하며 특정 분기 명령어의 개별성은 고려하지 않음
		  => 정적 예측
		- 동적 예측
			- 개별 분기 명령어의 행동에 의존하는 예측을 하며 프로그램이 진행되는 도중에 예측을 바꿀 수 있음
			- 각 분기가 일어났는지 안일어났는지 이력을 기록하고, 최근 과거 이력을 사용하여 미래를 예측함