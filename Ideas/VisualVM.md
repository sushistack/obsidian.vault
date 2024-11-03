
## 샘플러

샘플러만 사용해도 어디서 레이턴시가 발생하는지 확인이 가능하다.
이를 토대로 디버깅할 곳을 선정할 수 있다.

![](Pasted%20image%2020241101101619.png)
톰캣 실행 및 CPU 샘플링 시작 후, http 요청 후 끝날 때까지의 상황임

위 내용을 살펴 보면,

1. total time에 비해 CPU 사용 시간이 너무 적다 = CPU 작업이 아니라 I/O 작업을 하는데 시간을 소모했을 가능성이 크다.
2. SocketProcessorBase.run()에서 시간이 32초나 걸린 것으로 보아 네트워크 통신하는데 오래 걸린 것으로 추정이 가능하다.
3. getTask() 작업을 통해서 다음에 실행할 작업을 가져오는데에도 어느정도 시간이 필요한 것을 알 수 있다.


## 프로파일러

### CPU

![](Pasted%20image%2020241101113217.png)

위 처럼 CPU 시간과 invocations 수를 체크할 수 있다.


### JDBC

![](Pasted%20image%2020241101112938.png)
위 처럼 JDBC 드라이버에서 요청을 가로채서 확인이 가능하다.