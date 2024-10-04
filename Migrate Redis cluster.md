
### 사전 상황

- 구 레디스 클러스터
- 신 레디스 클러스터

각각 존재함

신 레디스 클러스터에서 노드들을 하나씩 제거해서 제거된 노드를 구 레디스 클러스터에 추가하는 방식으로 진행.

### 1. 신규 레디스 클러스터 노드 확인

```
cluster nodes 32fe43a6ff4b1f44c3bd8eb2f88309b6111fcd76 10.162.5.222:7105@17105 slave afbf7a2f8a752f8c2ae16b81539f3dfb89fddc31 0 1728001061000 3 connected 93998f21226d391af8d9fc7ac1f7b573dd0fd6a1 10.162.5.222:7106@17106 slave 415e9a81b3e13339b5fc48579e0e00f3eae1c8c0 0 1728001061986 2 connected 592b1618abb8d28a2a2e108563a71cf3267f80ff 10.162.5.222:7104@17104 slave 08f9c64a2cf044d246f0f9a9625aa22889335e1d 0 1728001061000 1 connected 08f9c64a2cf044d246f0f9a9625aa22889335e1d 10.162.5.222:7101@17101 myself,master - 0 1728001061000 1 connected 0-5460 afbf7a2f8a752f8c2ae16b81539f3dfb89fddc31 10.162.5.222:7103@17103 master - 0 1728001061585 3 connected 10923-16383 415e9a81b3e13339b5fc48579e0e00f3eae1c8c0 10.162.5.222:7102@17102 master - 0 1728001061081 2 connected 5461-10922
```

### 2. 클러스터 해제

```sh
$ redis-cli -h 127.0.0.1 -p 7101 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7102 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7103 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7104 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7105 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7106 cluster reset hard
```

####  데이터가 있는 상태에서 마스터 노드 삭제 시 다음과 같은 에러 발생

```
(error) ERR CLUSTER RESET can't be called with master nodes containing keys
```

Redis 클러스터에서 키가 남아 있는 마스터 노드에서는 바로 클러스터 초기화가 불가능하므로, 먼저 해당 데이터를 삭제한 후에 클러스터를 초기화해야 합니다.

#### slave 노드들 부터 삭제

```sh
$ redis-cli -h 127.0.0.1 -p 7104 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7105 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7106 cluster reset hard
```

```sh
$ redis-cli -h 127.0.0.1 -p 7104 cluster reset hard
OK
```


