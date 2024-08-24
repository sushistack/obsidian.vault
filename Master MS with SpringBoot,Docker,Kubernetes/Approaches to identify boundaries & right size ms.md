
![](Pasted%20image%2020240824164132.png)


## Domain-Driven Sizing

각 마이크로 서비스의 사이즈를 어떻게 정할 것인가?


## Event Storming Sizing



![](Pasted%20image%2020240824164510.png)

서비스를 어느정도까지 쪼갤 것이냐 ? 

![](Pasted%20image%2020240824165601.png)

![](Pasted%20image%2020240824165911.png)

각 ms 는 api gateway를 통해 요청을 받고, 받은 정보를 db등에 저장하면서 event bus에 데이터를 전송, 이벤트 버스를 통해 입력된 데이터는 다른 각 마이크로 서비스에 사용될 수 있게 전달된다.