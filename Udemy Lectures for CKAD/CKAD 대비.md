
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

```
$ k create namespace dev
$ k config set-context $(k config current-context) --namespace=dev
```

서비스 생성

```sh
kubectl expose <resource-type> <resource-name> [--port=<port>] [--target-port=<target-port>] [--type=<service-type>]
$ k expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

• <resource-type>: Service를 노출하려는 리소스의 유형 (예: pod, deployment, replicaset 등).
• <resource-name>: Service를 노출하려는 리소스의 이름.
• --port: Service가 외부에 노출할 포트 번호.
• --target-port: 실제 Pod에서 노출될 포트 번호 (디폴트는 --port와 동일).
• --type: Service의 유형 (ClusterIP, NodePort, LoadBalancer, ExternalName 중 하나).



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

