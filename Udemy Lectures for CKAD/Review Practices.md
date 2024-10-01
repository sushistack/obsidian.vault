
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


## 4. Namespace

```sh
$ k create namespace my-namespace
```

```sh
$ k [cmd] [kube-resource] -n [namespace]
```

```sh
$ k [cmd] [kube-resource] --all-namespaces
```


## 5. Imperative Commands

```sh
$ k run redis --labels tier=db
```

### Service

```sh
$ k expose -h
Expose a resource as a new Kubernetes service.
```

```sh
$ k run httpd --image=httpd:alpine
$ k expose po httpd --type=ClusterIP --name=httpd --target-port=80 --port=80
```

## 6. Docker Images 


```Dockerfile
FROM python:3.6
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt

ENTRYPOINT ["python", "app.py"] %% Fixed commands %%
CMD ["--color", "red"] %% overridable commands %%
```

## 7. Command and Arguments

```sh
$ docker run --name docker-image-name --color green
```

```yml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: docker-image-name
    command: ["python3", "app2.py"]
    args: ["--color", "green"]
```

## 8. ConfigMaps

```sh
$ k create cm configmap-name --from-literal="key1=value1" --from-literal="key2=value2" 
```

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-name
  namespace: default
data:
  key1: value1
  key2: value2
```

### 전체 로드

컨테이너 별로 참조

```diff
spec:
  containers:
+   - name: 
+     envFrom:
+       configMapRef:
+         name: configmap-name
```

### 일부만 타겟으로 가져오기

```diff
spec:
  containers:
+ - env:
+   - name: key1
+     value: value1
```

or

```diff
spec:
  containers:
+ - env:
+   - name: APP_COLOR
+     valueFrom:
+       configMapKeyRef:
+         name: configmap-name
+         key: key1
```
## 9. Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: default
data:
  DB_Host: c3FsMDE=
  DB_User: cm9vdA==
  DB_Password: cGFzc3dvcmQxMjM=
```

```diff
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
+   envFrom:
+   - secretRef:
+       name: db-secret
```

## 10. Security Context

### pods 전역으로 설정

```diff
spec:
+ securityContext:
+   runAsUser: 1010
```

### 컨테이너 별 설정

```diff
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
+    securityContext:
+     runAsUser: 1002
```

Pod 레벨 SC 보다 Container 레벨 SC가 우선 순위가 높다.
### Capabilities 설정

```diff
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
+    securityContext:
+     capabilities:
+       add: ["SYS_TIME"]
+       drop: ["MKNOD"]
```

**capabilities**는 Linux 커널의 특정 기능을 제어하여 루트 권한을 세분화하고, 불필요한 권한을 최소화하는 방식

## Service Account

애플리케이션(컨테이너화된 프로세스)나 자동화된 작업이 **Kubernetes API**와 상호작용할 수 있도록 설계된 계정

1. **API 서버와의 인증**
2. **권한 관리 (RBAC - Role-Based Access Control)**
3. **Pod에 할당되는 기본 인증 정보**
4. **애플리케이션의 권한 최소화**

Kubernetes에서는 **전역적으로 RBAC**로 권한을 설정하고, **세부적으로 Pod 단위로 Service Account(SA)**를 지정하는 방식이 일반적입니다. 이 접근 방식은 Kubernetes 클러스터에서 보안과 권한 관리를 효과적으로 수행하기 위한 좋은 설계 방법

### RBAC

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups:
  - ''
  resources:
  - pods
  verbs:
  - get
  - watch
  - list
```

### Role Binding

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: dashboard-sa # Name is case sensitive
  namespace: default
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```