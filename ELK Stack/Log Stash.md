> log(로그를) stash(저장한다)
> 플로그인 기반의 오픈소스 데이터 처리 파이프라인 도구
> 데이터를 저장하기 전에 원하는 형태로 가공하는 역할


## 특징

### 1. 플러그인 기반

: 파이프라인을 구성하는 각 요소들은 전부 플러그인 형태로 만들어져 있다.  
(플러그인 생성 방법 [contributing-to-logstash](https://www.elastic.co/guide/en/logstash/current/contributing-to-logstash.html))

### 2. 모든 형태의 데이터 처리

JSON, XML 등의 다양한 형태의 데이터를 입력받아 가공한 다음 저장할 수 있다.

### 3. 성능

자체적으로 내장되어 있는 메모리와 파일 기반의 큐를 사용하므로 처리 속도와 안정성이 높다.

### 4. 안정성

장애 상황에 대응하기 위한 재시도 로직이나 오류가 발생한 도큐먼트를 따로 보관하는 [dead-letter-queue](https://www.elastic.co/guide/en/logstash/current/dead-letter-queues.html)를 내장하고 있다.