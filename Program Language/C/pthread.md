- pthread_create
	```C
int pthread_create(pthread_t* thread, const pthread_attr_t* attr, void* (*start_routine) (void *), void* arg)
```
	- 현재 실행되고 있는 프로세스에서 새로운 스레드 생성
	- 새로운 스레드는 argument로 전달된 start_routing() 함수를 실행함
	- parameter
		1. thread: 새로 생성된 thread ID가 저장됨
			- 이 인자로 넘어온 값을 통해서 pthread_join과 같은 함수를 사용할 수 있음
		2. attr: 스레드의 특성을 정의(기본적으로 NULL 지정)
			- 만약 스레드의 속성을 지정하려고 한다면, pthread_attr_init등의 함수로 초기화해야함
		3. start_routine: 새로운 스레드가 실행할 함수 포인터
			- 이 함수는 스레드가 시작되면 호출, 스레드의 작업을 정의하는 역할
		4. arg: start_routine에 전달될 인자.
	- return값
		- 성공: `return 0`
		- 실패: `return errno`
			- 좀비 스레드가 되고, 이 좀비 스레드는 자원을 소모하게 되어 더 이상 스레드를 생성할 수 없게 됨
- pthread_detach
	
	```C
int pthread_detach(pthread_t thread)
```
	- 해당 스레드가 종료될 때 자동으로 자원을 해제하도록 하는 역할
	- 메인 스레드가 명시적으로 대기하지 않고도 스레드가 종료될 때 자동으로 리소스를 회수할 수 있음
	- 일반적으로 스레드를 생성한 후, `detach`함수를 호출하여 해당 스레드를 분리함
	- 스레드가 분리되면 해당 스레드는 종료될 때 자동으로 리소스를 해제하며, 메인 스레드는 해당 스레드의 종료를 대기하지 않고 다른 작업을 수행할 수 있음
	- parameter
		1. thread: 분리할 thread ID
	- return값
		- 성공: `return 0`
		- 실패: `return errno`
	- pthread_detach와 pthread_join을 동시에 사용할 수는 없음