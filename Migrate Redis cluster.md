
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
/usr/local/redis6/bin/redis-server ~/redis-cluster/7101/redis.conf
```

실행하여, 신규 우분투 서버에 redis6 의 redis cluster 1대를 띄움.

```
$ redis-cli -h 10.162.5.39 -p 7000 cluster meet 10.162.5.222 7101
```

위 명령어를 실행하여 클러스터에 redis6 버전의 신규 노드를 추가함.


```
-wb801:~/redis-cluster$ redis-cli -p 7101 cluster nodes                          
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728278971000 0 connected
8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave 42012ea854090f6ded8ca52a5b2d71a37567baee 0 1728278971594 9 connected             f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728278970587 5 connected 305-432 10923-11255                                    42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728278970286 9 connected 5461-10922                                             8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728278971292 3 connected 13300-16383                                            81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 master - 0 1728278970789 8 connected 0-304 433-5460 11256-13299
```

정상적으로 연결되었다.


16,384 / 6 = 2730.6666 = 약2732으로 슬럿을 분리하도록 하겠다.

리샤딩을 진행한다. 

```
irteam@cupw-alpha-wb801:~/redis-cluster$ /usr/local/redis6/bin/redis-cli --cluster reshard 10.162.5.222:7101  
        >>> Performing Cluster Check (using node 10.162.5.222:7101)  
M: 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101  
slots: (0 slots) master  
S: 8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004  
slots: (0 slots) slave  
replicates 42012ea854090f6ded8ca52a5b2d71a37567baee  
M: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003  
slots:[305-432],[10923-11255] (461 slots) master  
M: 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001  
slots:[5461-10922] (5462 slots) master  
   1 additional replica(s)  
M: 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002  
slots:[13300-16383] (3084 slots) master  
M: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000  
slots:[0-304],[433-5460],[11256-13299] (7377 slots) master  
[OK] All nodes agree about slots configuration.  
        >>> Check for open slots...  
        >>> Check slots coverage...  
        [OK] All 16384 slots covered.  
        How many slots do you want to move (from 1 to 16384)? 2732  
        What is the receiving node ID? 9020d0fb028ae9ad2b6e30a6e61ce2328c874670  
        Please enter all the source node IDs.  
        Type 'all' to use all the nodes as source nodes for the hash slots.  
        Type 'done' once you entered all the source nodes IDs.  
        Source node #1: all  
  
        Ready to move 2732 slots.  
        Source nodes:  
        M: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003  
        slots:[305-432],[10923-11255] (461 slots) master  
        M: 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001  
        slots:[5461-10922] (5462 slots) master  
        1 additional replica(s)  
        M: 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002  
        slots:[13300-16383] (3084 slots) master  
        M: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000  
        slots:[0-304],[433-5460],[11256-13299] (7377 slots) master  
        Destination node:  
        M: 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101  
        slots: (0 slots) master
```


```
wb801:~/redis-cluster$ /usr/local/redis6/bin/redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728279596000 10 connected 0-380 433-1358 5461-6370 13300-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave 42012ea854090f6ded8ca52a5b2d71a37567baee 0 1728279596156 9 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728279597162 5 connected 381-432 10923-11255  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728279596659 9 connected 6371-10922  
        8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728279597061 3 connected 13814-16383  
        81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 master - 0 1728279596558 8 connected 1359-5460 11256-13299
```

리샤딩 잘 되었다.

이제 노드 하나를 삭제해보자


```
irteam@cupw-alpha-wb801:~/redis-cluster$ /usr/local/redis6/bin/redis-cli --cluster reshard 10.162.5.222:7101  
        >>> Performing Cluster Check (using node 10.162.5.222:7101)  
M: 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101  
slots:[0-380],[433-1358],[5461-6370],[13300-13813] (2731 slots) master  
S: 8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004  
slots: (0 slots) slave  
replicates 42012ea854090f6ded8ca52a5b2d71a37567baee  
M: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003  
slots:[381-432],[10923-11255] (385 slots) master  
M: 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001  
slots:[6371-10922] (4552 slots) master  
   1 additional replica(s)  
M: 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002  
slots:[13814-16383] (2570 slots) master  
M: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000  
slots:[1359-5460],[11256-13299] (6146 slots) master  
[OK] All nodes agree about slots configuration.  
        >>> Check for open slots...  
        >>> Check slots coverage...  
        [OK] All 16384 slots covered.  
        How many slots do you want to move (from 1 to 16384)? 6146  
        What is the receiving node ID? 9020d0fb028ae9ad2b6e30a6e61ce2328c874670  
        Please enter all the source node IDs.  
        Type 'all' to use all the nodes as source nodes for the hash slots.  
        Type 'done' once you entered all the source nodes IDs.  
        Source node #1: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a  
        Source node #2: done

```

```
Node 10.162.5.39:7000 replied with error:  
ERR Please use SETSLOT only with masters.  
        irteam@cupw-alpha-wb801:~/redis-cluster$ /usr/local/redis6/bin/redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728280596000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave 42012ea854090f6ded8ca52a5b2d71a37567baee 0 1728280598000 9 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728280598408 5 connected 381-432 10923-11255  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728280597905 9 connected 6371-10922  
        8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728280599011 3 connected 13814-16383  
        81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 slave 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 0 1728280598910 10 connected
```

슬럿 = 0 이되니 slave가 되었다.

### 2, 3번째 노드도 띄운다.


```
/usr/local/redis6/bin/redis-server ~/redis-cluster/7102/redis.conf
/usr/local/redis6/bin/redis-server ~/redis-cluster/7103/redis.conf
```


```sh
alpha-wb801:~/redis-cluster$ ps -ef | grep redis  
irteam   3758519       1  0 14:26 ?        00:00:10 /usr/local/redis6/bin/redis-server 0.0.0.0:7101 [cluster]  
irteam   3793562       1  0 15:00 ?        00:00:00 /usr/local/redis6/bin/redis-server *:7102 [cluster]  
irteam   3793771       1  0 15:00 ?        00:00:00 /usr/local/redis6/bin/redis-server *:7103 [cluster]
```

미팅을 주선한다.

```sh
# 우분투 서버에서 진행
/usr/local/redis6/bin/redis-cli -p 7101 cluster meet 10.162.5.222 7102
/usr/local/redis6/bin/redis-cli -p 7101 cluster meet 10.162.5.222 7103
```


```sh
$ /usr/local/redis6/bin/redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728280971000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave 42012ea854090f6ded8ca52a5b2d71a37567baee 0 1728280971601 9 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728280972607 5 connected 381-432 10923-11255  
        967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728280972506 11 connected  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728280972506 0 connected  
42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728280971601 9 connected 6371-10922  
        8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728280972103 3 connected 13814-16383  
        81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 slave 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 0 1728280972000 10 connected
```

2대도 잘 붙었다.

### 리샤딩

```sh
$ /usr/local/redis6/bin/redis-cli --cluster reshard 10.162.5.222:7101  
        >>> Performing Cluster Check (using node 10.162.5.222:7101)  
M: 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101  
slots:[0-380],[433-6370],[11256-13813] (8877 slots) master  
   1 additional replica(s)  
S: 8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004  
slots: (0 slots) slave  
replicates 42012ea854090f6ded8ca52a5b2d71a37567baee  
M: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003  
slots:[381-432],[10923-11255] (385 slots) master  
M: 967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103  
slots: (0 slots) master  
M: c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102  
slots: (0 slots) master  
M: 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001  
slots:[6371-10922] (4552 slots) master  
   1 additional replica(s)  
M: 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002  
slots:[13814-16383] (2570 slots) master  
S: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000  
slots: (0 slots) slave  
replicates 9020d0fb028ae9ad2b6e30a6e61ce2328c874670  
[OK] All nodes agree about slots configuration.  
        >>> Check for open slots...  
        >>> Check slots coverage...  
        [OK] All 16384 slots covered.  
        How many slots do you want to move (from 1 to 16384)? 4552  
        What is the receiving node ID? c2219f3543e03fb5b3ddc128f677c5fe87ee1e09  
        Please enter all the source node IDs.  
        Type 'all' to use all the nodes as source nodes for the hash slots.  
        Type 'done' once you entered all the source nodes IDs.  
        Source node #1: 42012ea854090f6ded8ca52a5b2d71a37567baee  
        Source node #2: done
```



```sh
$ /usr/local/redis6/bin/redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728281438000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 0 1728281439474 12 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728281439575 5 connected 381-432 10923-11255  
        967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728281439000 11 connected  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728281439574 12 connected 6371-10922  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728281439575 9 connected  
8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 master - 0 1728281439000 3 connected 13814-16383  
        81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 slave 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 0 1728281440077 10 connected
```


```sh
$ /usr/local/redis6/bin/redis-cli --cluster reshard 10.162.5.222:7101  
        >>> Performing Cluster Check (using node 10.162.5.222:7101)  
M: 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101  
slots:[0-380],[433-6370],[11256-13813] (8877 slots) master  
   1 additional replica(s)  
S: 8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004  
slots: (0 slots) slave  
replicates c2219f3543e03fb5b3ddc128f677c5fe87ee1e09  
M: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003  
slots:[381-432],[10923-11255] (385 slots) master  
M: 967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103  
slots: (0 slots) master  
M: c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102  
slots:[6371-10922] (4552 slots) master  
   1 additional replica(s)  
M: 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001  
slots: (0 slots) master  
M: 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002  
slots:[13814-16383] (2570 slots) master  
S: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000  
slots: (0 slots) slave  
replicates 9020d0fb028ae9ad2b6e30a6e61ce2328c874670  
[OK] All nodes agree about slots configuration.  
        >>> Check for open slots...  
        >>> Check slots coverage...  
        [OK] All 16384 slots covered.  
        How many slots do you want to move (from 1 to 16384)? 2570  
        What is the receiving node ID? 967b110d370b6f38077ccec78968e206bb2ba91a  
        Please enter all the source node IDs.  
        Type 'all' to use all the nodes as source nodes for the hash slots.  
        Type 'done' once you entered all the source nodes IDs.  
        Source node #1: 8ce0a48d33af04962114460baad4c46f860fd57c  
        Source node #2: done
```


```sh
$ /usr/local/redis6/bin/redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728281662000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 0 1728281663735 12 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728281663232 5 connected 381-432 10923-11255  
        967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728281663534 13 connected 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728281663000 12 connected 6371-10922  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728281663031 9 connected  
8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 slave 967b110d370b6f38077ccec78968e206bb2ba91a 0 1728281662629 13 connected  
81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 slave 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 0 1728281663534 10 connected
```


### 슬럿 제거

```sh
$ /usr/local/redis6/bin/redis-cli --cluster reshard 10.162.5.222:7101  
        >>> Performing Cluster Check (using node 10.162.5.222:7101)  
M: 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101  
slots:[0-380],[433-6370],[11256-13813] (8877 slots) master  
   1 additional replica(s)  
S: 8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004  
slots: (0 slots) slave  
replicates c2219f3543e03fb5b3ddc128f677c5fe87ee1e09  
M: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003  
slots:[381-432],[10923-11255] (385 slots) master  
M: 967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103  
slots:[13814-16383] (2570 slots) master  
   1 additional replica(s)  
M: c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102  
slots:[6371-10922] (4552 slots) master  
   1 additional replica(s)  
M: 42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001  
slots: (0 slots) master  
S: 8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002  
slots: (0 slots) slave  
replicates 967b110d370b6f38077ccec78968e206bb2ba91a  
S: 81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000  
slots: (0 slots) slave  
replicates 9020d0fb028ae9ad2b6e30a6e61ce2328c874670  
[OK] All nodes agree about slots configuration.  
        >>> Check for open slots...  
        >>> Check slots coverage...  
        [OK] All 16384 slots covered.  
        How many slots do you want to move (from 1 to 16384)? 385  
        What is the receiving node ID? 967b110d370b6f38077ccec78968e206bb2ba91a  
        Please enter all the source node IDs.  
        Type 'all' to use all the nodes as source nodes for the hash slots.  
        Type 'done' once you entered all the source nodes IDs.  
        Source node #1: f717e501df1e87e4e98c9dd0ff0aa5db8398fc36  
        Source node #2: done
```

```
$ redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728284776000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 0 1728284777910 12 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728284778111 5 connected  
967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728284777000 13 connected 381-432 10923-11255 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728284777000 12 connected 6371-10922  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728284777000 9 connected  
8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 slave 967b110d370b6f38077ccec78968e206bb2ba91a 0 1728284778011 13 connected  
81e57f3e4fe49604d40b63ad4c3fc124af130f4a 10.162.5.39:7000@17000 slave 9020d0fb028ae9ad2b6e30a6e61ce2328c874670 0 1728284777609 10 connected
```

```sh
# 7000
/usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 81e57f3e4fe49604d40b63ad4c3fc124af130f4a

# 7002
/usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 8ce0a48d33af04962114460baad4c46f860fd57c

# 7004
/usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 8db916dabb8ad8d1130da6821ec2e4857e7e0606
```



```sh
$ /usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 81e57f3e4fe49604d40b63ad4c3fc124af130f4a  
>>> Removing node 81e57f3e4fe49604d40b63ad4c3fc124af130f4a from cluster 10.162.5.222:7101  
        >>> Sending CLUSTER FORGET messages to the cluster...  
        >>> Sending CLUSTER RESET SOFT to the deleted node.  

$ redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728285025000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 0 1728285026844 12 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728285026000 5 connected  
967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728285026040 13 connected 381-432 10923-11255 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728285026341 12 connected 6371-10922  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728285026542 9 connected  
8ce0a48d33af04962114460baad4c46f860fd57c 10.162.5.39:7002@17002 slave 967b110d370b6f38077ccec78968e206bb2ba91a 0 1728285026000 13 connected
```


```sh
$ /usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 8ce0a48d33af04962114460baad4c46f860fd57c  
>>> Removing node 8ce0a48d33af04962114460baad4c46f860fd57c from cluster 10.162.5.222:7101  
        >>> Sending CLUSTER FORGET messages to the cluster...  
        >>> Sending CLUSTER RESET SOFT to the deleted node.  

$ redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728285108000 10 connected 0-380 433-6370 11256-13813  
        8db916dabb8ad8d1130da6821ec2e4857e7e0606 10.162.5.39:7004@17004 slave c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 0 1728285109580 12 connected  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728285109000 5 connected  
967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728285109078 13 connected 381-432 10923-11255 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728285108777 12 connected 6371-10922  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728285109000 9 connected
```


```sh
$ /usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 8db916dabb8ad8d1130da6821ec2e4857e7e0606  
>>> Removing node 8db916dabb8ad8d1130da6821ec2e4857e7e0606 from cluster 10.162.5.222:7101  
        >>> Sending CLUSTER FORGET messages to the cluster...  
        >>> Sending CLUSTER RESET SOFT to the deleted node.  


$ redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728285160000 10 connected 0-380 433-6370 11256-13813  
f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 10.162.5.39:7003@17003 master - 0 1728285161021 5 connected  
967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728285161522 13 connected 381-432 10923-11255 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728285161523 12 connected 6371-10922  
        42012ea854090f6ded8ca52a5b2d71a37567baee 10.162.5.39:7001@17001 master - 0 1728285160520 9 connected
```

```sh
$ /usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 42012ea854090f6ded8ca52a5b2d71a37567baee
```

```sh
$ /usr/local/redis6/bin/redis-cli --cluster del-node 10.162.5.222:7101 f717e501df1e87e4e98c9dd0ff0aa5db8398fc36  
>>> Removing node f717e501df1e87e4e98c9dd0ff0aa5db8398fc36 from cluster 10.162.5.222:7101  
        >>> Sending CLUSTER FORGET messages to the cluster...  
        >>> Sending CLUSTER RESET SOFT to the deleted node.  


```

신규 노드만 존재함을 확인

```
$ redis-cli -p 7101 cluster nodes  
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728285462000 10 connected 0-380 433-6370 11256-13813  
        967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728285462577 13 connected 381-432 10923-11255 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728285463580 12 connected 6371-10922
```


![](Pasted%20image%2020241007161646.png)

### redis 7 버전 노드 생성

```sh
$ redis-server ~/redis-cluster/7104/redis.conf
$ redis-server ~/redis-cluster/7105/redis.conf
$ redis-server ~/redis-cluster/7106/redis.conf
```

```diff
$ ps -ef | grep redis  
irteam   3758519       1  0 14:26 ?        00:00:26 /usr/local/redis6/bin/redis-server 0.0.0.0:7101 [cluster]  
irteam   3793562       1  0 15:00 ?        00:00:24 /usr/local/redis6/bin/redis-server *:7102 [cluster]  
irteam   3793771       1  0 15:00 ?        00:00:11 /usr/local/redis6/bin/redis-server *:7103 [cluster]  
+irteam   3895006       1  0 16:19 ?        00:00:00 redis-server *:7104 [cluster]  
+irteam   3895756       1  0 16:20 ?        00:00:00 redis-server *:7105 [cluster]  
+irteam   3895966       1  0 16:20 ?        00:00:00 redis-server *:7106 [cluster]
```


```sh
$ /usr/local/redis6/bin/redis-cli -p 7101 cluster meet 10.162.5.222 7104
$ /usr/local/redis6/bin/redis-cli -p 7101 cluster meet 10.162.5.222 7105
$ /usr/local/redis6/bin/redis-cli -p 7101 cluster meet 10.162.5.222 7106
```


redis6 클러스터에 redis7 노드가 추가 되지 않는다.

redis6 신규 노드로 추가한 뒤, 노드 연결을 시킨 후, redis6 신규 노드를 redis7 신규 노드로 새로 띄운다.

```
/usr/local/redis6/bin/redis-cli -p 7104 cluster meet 10.162.5.222 7104
```

정상 연결되었다.
```sh
9020d0fb028ae9ad2b6e30a6e61ce2328c874670 10.162.5.222:7101@17101 myself,master - 0 1728288607000 10 connected 0-380 433-6370 11256-13813  
        4ea766f257e5d41c34c252c9ce62913c68bfd416 10.162.5.222:7104@17104 master - 0 1728288607246 0 connected  
967b110d370b6f38077ccec78968e206bb2ba91a 10.162.5.222:7103@17103 master - 0 1728288607000 13 connected 381-432 10923-11255 13814-16383  
c2219f3543e03fb5b3ddc128f677c5fe87ee1e09 10.162.5.222:7102@17102 master - 0 1728288607747 12 connected 6371-10922
```

redis 6.x -> 7.2 는 호환성이 완벽하지 않다고함.
6.x -> 7.0 -> 7.2 순으로 가야한다고 함.
6.x -> 7.0 -> 7.4 순으로 진행


```sh
irteam@cupw-alpha-wb801:~$ cd /home1/irteam/apps
irteam@cupw-alpha-wb801:~$ wget https://github.com/redis/redis/archive/refs/tags/7.0.15.tar.gz


make -> c 권한 실패
make distclean -> make 캐시 삭제
sudo make
sudo make PREFIX=/usr/local/redis7.0 install
```

```sh
/usr/local/redis7.0/bin$ ls 
redis-benchmark  redis-check-aof  redis-check-rdb  redis-cli  redis-sentinel  redis-server
```

설치 완료

redis7.0 node 실행

```
/usr/local/redis7.0/bin/redis-server ~/redis-cluster/7104/redis.conf
/usr/local/redis7.0/bin/redis-server ~/redis-cluster/7105/redis.conf
/usr/local/redis7.0/bin/redis-server ~/redis-cluster/7106/redis.conf
```