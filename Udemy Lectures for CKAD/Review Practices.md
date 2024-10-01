
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

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
  labels:
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-container
          image: nginx
```