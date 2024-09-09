
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

• resource-type: Service를 노출하려는 리소스의 유형 (예: pod, deployment, replicaset 등).
• resource-name: Service를 노출하려는 리소스의 이름.
• --port: Service가 외부에 노출할 포트 번호.
• --target-port: 실제 Pod에서 노출될 포트 번호 (디폴트는 --port와 동일).
• --type: Service의 유형 (ClusterIP, NodePort, LoadBalancer, ExternalName 중 하나)

### Command & Arguments

```sh
CMD ["sleep", "5"]
# or
CMD sleep 5
# or
ENTRYPOINT ["sleep"]
CMD ["5"]
```

ENTRYPOINT: 고정형 명령어 또는 인수 사용
CMD: 동적 명령어 또는 인수 전달

kube : docker
commands : ENTRYPOINT
args: CMD

### ConfigMaps

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  key1: value1
  key2: value2
```
  
```yml
envFrom:
	- configMapRef:
		name: app-config
```

```yml
env:
	- name: APP_COLOR
	  valueFrom:
		  configMapKeyRef:
			  name: app-config
			  key: APP_COLOR
volumes:
	- name: app-config-volume
	  configMap:
		  name: app-config
```

### Secret

```yml
apiVersion: v1
kind: Secret
metadata: 
	name: app-secret
data:
	DB_Host: mysql
```

```sh
$ echo -n 'password' | base64
```

**기본 요구 사항**: Kubernetes의 Secret 데이터는 base64로 인코딩하여 Secret 객체의 data 필드에 저장합니다. base64 인코딩은 데이터의 이진 값을 ASCII 문자열로 변환하는 방식입니다.


```yml
spec:
	containers
		- name: web-app
		  envFrom:
			  -secretRef:
				  name: app-secret

volumes:
	- name: app-secret-volume
	  secret:
		  secretName: app-secret
```

### Security Context

```bash
$ kubectl exec ubuntu-sleeper -- whoami
$ kubectl exec ubuntu-sleeper --user=1010
```

#### 컨테이너들 전체에 대한 유저


```yml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
```

#### 특정 컨테이너에 대한 유저 및 능력 부여

```yml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002
      capabilities:
        add: ["SYS_TIME"]

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
```

### Service Account

