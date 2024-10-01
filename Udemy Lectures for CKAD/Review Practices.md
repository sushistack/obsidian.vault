
## 1. Pods

```sh
$ k run redis-pod \
--image=redis \
--port=6291 \
--env="SAMPLE_KEY1=sampleValue1" \
--env="SAMPLE_KEY2=samplevalue2" \
--labels="app=redis,env=dev" \
--dry-run=client \
-o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: redis
    env: dev
  name: redis-pod
spec:
  containers:
  - env:
    - name: SAMPLE_KEY1
      value: sampleValue1
    - name: SAMPLE_KEY2
      value: samplevalue2
    image: redis
    name: redis-pod
    ports:
    - containerPort: 6291
    resources: {}
  dnsPolicy: ClusterFirst # ClusterFirstWithHostNet
  restartPolicy: Always
status: {}
```

**ClusterFirstWithHostNet**: 
1. **네트워크 성능을 위해 호스트 네트워크를 사용**
2. **호스트 네트워크에서 클러스터 내부 서비스와 통신**
3. **Pod가** **hostNetwork: true** 로 설정


## 2. ReplicaSets

생성 명령어 없음.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
  labels: # 이 레이블은 다른 리소스가 이 ReplicaSet을 찾는데 사용 됨.
    type: replicaset
    name: rs-nginx
spec:
  replicas: 3
  selector:  # 이 레이블과 매칭되는 Pod 만 관리.
    matchLabels:
      app: nginx-app
  template:
    metadata:
      name: nginx-pod
      labels: # Pod의 Label
        app: nginx-app
    spec:
      containers:
        - name: nginx-container
          image: nginx
```
### 간단한 스케일링

```sh
$ k scale --replicas=5 rs <replicaset-name>
```

## 3. Deployments

```sh
$ k create deploy \
my-deployment \
--image=nginx \
# --image=redis \ 멀티 컨테이너 설정 시에는 개별 포트 설정 불가
--replicas=3 \
--port=80 \
--dry-run=client \
-o yaml
```


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-deployment  # 자동으로 붙었다
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
      - image: redis
        name: redis
        resources: {}
status: {}
```


## Namespace

```sh
$ k create namespace my-namespace
```

```sh
$ k [cmd] [kube-resource] -n [namespace]
```

```sh
$ k [cmd] [kube-resource] --all-namespaces
```


## Imperative Commands

```sh
$ k [cmd] [kube-resource] --labels tier=db
```
