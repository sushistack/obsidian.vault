
```
$ k run nginx --image=nginx
$ k create -f pod.yml
$ k create replicaset my-replicaset --image=nginx --replicas=3
$ k create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```

deployment
ㄴ replicaset
	ㄴ pod1
	ㄴ pod2
	ㄴ pod3

Deployment는 전반적인 배포 전략을 관리하고, ReplicaSet은 해당 전략에 따라 실제 Pod를 관리하는 역할을 맡습니다.








---


## 모의 시험

Create a pod called `time-check` in the `dvl1987` namespace. This pod should run a container called `time-check` that uses the `busybox` image.

1. Create a config map called `time-config` with the data `TIME_FREQ=10` in the same namespace.
2. The `time-check` container should run the command: `while true; do date; sleep $TIME_FREQ;done` and write the result to the location `/opt/time/time-check.log`.
3. The path `/opt/time` on the pod should mount a volume that lasts the lifetime of this pod.


### 1. time-config

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
data:
  TIME_FREQ: '10'
```



## Commands

