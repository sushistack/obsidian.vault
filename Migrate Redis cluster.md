
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

#### master 노드 삭제를 위한 모든 키 제거

```sh
$ redis-cli -h 127.0.0.1 -p 7101 FLUSHALL
$ redis-cli -h 127.0.0.1 -p 7102 FLUSHALL
$ redis-cli -h 127.0.0.1 -p 7103 FLUSHALL
```

```sh
$ redis-cli -h 127.0.0.1 -p 7101 FLUSHALL
OK
```

```
$ redis-cli -h 127.0.0.1 -p 7101 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7102 cluster reset hard
$ redis-cli -h 127.0.0.1 -p 7103 cluster reset hard
```

#### 마스터 노드 클러스터에서 제거

```
$ redis-cli -h 127.0.0.1 -p 7101 cluster reset hard
OK
```


#### 클러스터 확인

```
cluster nodes d84d733729d83a2dc5810470bed4fed87a83744c 10.162.5.222:7102@17102 myself,master - 0 1728003804000 0 connected
```

각 노드 별로 개별 클러스터로 떠 있는 것으로 확인

### 3. 각 레디스 노드 종료

#### 종료 및 systemd 중지

```
sudo systemctl stop redis && sudo systemctl disable redis && sudo systemctl status redis
```

#### 클러스터 정보 제거

```sh
$ rm 7101/nodes.conf
```


### 4. 구 레디스 클러스터에 신규 노드 추가

#### 사전 클러스터 정보

```
> cluster nodes 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728005236000 7 connected 5795-10922 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 master - 0 1728005235797 1 connected 433-5460 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728005236800 3 connected 11256-16383 f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 myself,master - 0 1728005236000 5 connected 0-432 5461-5794 10923-11255 8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave 42012ea854090f6ded8ca52a5b2d71a37567baee 0 1728005235000 7 connected
```


### 1. 신규 노드 생성 및 구 레디스 클러스터에 추가

#### 신규 레디스 노드 시작

```
$ redis-server ~/redis-cluster/7101/redis.conf

$ ps -ef | grep redis
redis-server *:7101 [cluster]
```

#### 새 노드를 기존 클러스터에 추가

```sh
# 기존 레디스 클러스터 서버에서
$ redis-cli -h 10.162.5.39 -p 7000 cluster meet 10.162.5.222 7101

```

```
redis-cli --cluster add-node 10.162.5.222:7101 10.162.5.39:7000 --cluster-slave --cluster-master-id 81e57f3e4fe49604d40b63ad4c3fc124af130f4a

redis-cli --cluster add-node 10.162.5.222:7102 10.162.5.39:7001 --cluster-slave --cluster-master-id 8ce0a48d33af04962114460baad4c46f860fd57c


```

=> 노드 추가가 안됨.
#### 슬롯 확인


```
$ redis-cli -p 7000 cluster nodes
8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728008276574 3 connected 11256-16383
8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave 42012ea854090f6ded8ca52a5b2d71a37567baee 0 1728008277576 7 connected
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728008277000 5 connected 0-432 5461-5794 10923-11255
42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728008277876 7 connected 5795-10922
81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 myself,master - 0 1728008276000 1 connected 433-5460
```


```
$ redis-cli --cluster fix 10.162.5.39:7003

>>> Fixing open slot 5693
*** Found keys about slot 5693 in non-owner node 10.162.5.39:7001!
Set as importing in: 10.162.5.39:7001
>>> Nobody claims ownership, selecting an owner..
```


```
redis-cli --cluster add-node 10.162.5.222:7103 10.162.5.222:7101 --cluster-slave --cluster-master-id 46ab98d0e8f9d5e9ea7b2419a0af711f0d08a740
```

### 미해결

https://github.com/redis/redis/issues/12685

미해결 이슈로 남아 있고, 단계적으로 업그레이드를 해야할 것으로 보임


6.2.14 버전을 설치하기로 함

```sh
irteam@cupw-alpha-wb801:~$ cd /home1/irteam/apps
irteam@cupw-alpha-wb801:~$ wget https://github.com/redis/redis/archive/refs/tags/6.2.14.tar.gz
make -> c 권한 실패
make distclean -> make 캐시 삭제
sudo make
sudo make PREFIX=/usr/local/redis6 install
```

```

```
