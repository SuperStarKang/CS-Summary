- 운영체제 입장에서 작업의 최소 단위는 [[Process]]
- [[CPU]] 입장에서 작업의 최소 단위는 [[Thread]]
#### Context란?
- CPU가 하나의 프로세스(또는 스레드)를 실행하는 동안 유지해야 하는 모든 정보
	- 현재 실행 중인 프로세스(또는 스레드)의 상태(State)
		1. CPU Registers(PC, ...)
		2. 메모리 관련 정보(페이지 테이블)
		3. 프로세스 관련 정보(PID, FD, ...)
		4. 스레드 관련 정보(스레드 레지스터 상태, 스택)
#### Context Switching
- 작업의 주체가 현재 context를 잠시 중단하고 다른 context를 실행하는 것
- Process Context Switching: 프로세서가 지금까지 실행되던 프로세스를 중단하고 다른 프로세스의 PCB 정보를 바탕으로 실행하는 것
- Thread Context Switching: 동일한 프로세스 내에서 하나의 스레드를 중지하고 다른 스레드의 TCB 정보를 바탕으로 스레드를 실행하는 것
- Process는 하나 이상의 Thread로 동작하기 때문에 Context Switching의 최소 단위는 TCB