
## API Resource

```sh
$ kubectl api-resources
```


## Practice 1 - Pods

### How many `pods` exist on the system?

```sh
$ k get pods
```

### Create a new pod with the `nginx` image.

```sh
$ k run nginx-app --image=nginx
```

### What is the image used to create the new pods?

```sh
$ k describe pods newpods-sqbl9
```

### Which nodes are these pods placed on?

```sh
$ k describe pods nginx-app | grep Node
```

### How many containers are part of the pod `webapp`?

```sh
$ k get pods
```

### What images are used in the new `webapp` pod?

```sh
$ k describe pods webapp | grep Image
```

### What is the state of the container `agentx` in the pod `webapp`?

```sh
$ k describe pods webapp | grep -i state
```

### Why do you think the container `agentx` in pod `webapp` is in error?

```
Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

### What does the `READY` column in the output of the `kubectl get pods` command indicate?

=> `{{ Running Containers }}  /  {{ Total Containers }}`

### Delete the `webapp` Pod.

```sh
$ k delete pods webapp
```

### Create a new pod with the name `redis` and the image `redis123`.

```sh
$ k run redis --image=redis123 --dry-run=client -o yaml > redis-definition.yaml
```

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis123
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

or

```sh
$ k run redis --image=redis123
```

### Now change the image on this pod to `redis`.

```sh
$ k create -f redis-definition.yaml
```

## Practice 2 - ReplicaSets

### How many ReplicaSets exist on the system?

```sh
$ k get replicasets
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       6s
```

### What is the image used to create the pods in the `new-replica-set`?

```sh
$ k describe replicasets new-replica-set
```

### How many PODs are READY in the `new-replica-set`?

```sh
$ k get replicasets
```

### Why do you think the PODs are not ready?

```sh
$ k describe pods new-replica-set-lrvnw
Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

### Delete Pod of 4 pods and How many PODs exist now?

```sh
controlplane ~ ➜  k get pods
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-f6cjz   0/1     ImagePullBackOff   0          6m
new-replica-set-bttmh   0/1     ImagePullBackOff   0          6m
new-replica-set-wkhnm   0/1     ErrImagePull       0          6m
new-replica-set-8wkcp   0/1     ImagePullBackOff   0          14s
```

=> Replicaset ensures that desired number of Pods always run


### Create a ReplicaSet using the `replicaset-definition-1.yaml` file located at `/root/`.

```sh
$ k create -f replicaset-definition-1.yaml 
error: resource mapping not found for name: "replicaset-1" namespace: "" from "replicaset-definition-1.yaml": no matches for kind "ReplicaSet" in version "v1"
ensure CRDs are installed first
```

=> ReplicaSet 은 **apiVersion: apps/v1**

### Fix the issue in the `replicaset-definition-2.yaml` file and create a `ReplicaSet` using it.

```yml
spec:
	replicas: 3 
	selector: 
		matchLabels: 
			app: my-app
			tier: nginx
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: nginx
		spec:
			containers:
				- name: nginx-container
				  image: nginx
```

=> `spec.selector.matchLabels.레이블명` 과 `spec.template.metadata.labels.레이블명` 은 완벽히 일치해야 함. (해당 Pods 들이 해당 ReplicaSet에 의해 관리되고 있음을 명확히 할 수 있음)

### Delete the two newly created ReplicaSets - `replicaset-1` and `replicaset-2`


```sh
$ k delete rs replicaset-1
```

### Fix the original replica set `new-replica-set` to use the correct `busybox` image.

#### 내가 만든 정답

```sh
$ k get rs new-replica-set -o yaml > new-replicasets.yaml
# new-replicasets.yaml  수정
$ k delete rs new-replica-set
$ k create -f new-replicasets.yaml
```

#### 정답

```sh
$ k edit rs new-replica-set
```

#### 참고

```sh
$ k replace -f new-replicasets.yaml
```
이건 왜 안됐을까?


### Scale the ReplicaSet to 5 PODs.

```sh
$ k scale --replicas=5 rs new-replica-set
```


## Practice  3 - Deployments

### How many Deployments exist on the system?

```sh
$ k get deploy
```

### What is images of Pods

```sh
$ k describe deployments frontend-deployment
```

### Create a new Deployment using the `deployment-definition-1.yaml` file located at `/root/`.

Deployment 는 apps/v1


### Create a new Deployment with the below attributes using your own deployment definition file.


```sh
$ k create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```


## Practice 4 - Namespace

### How many Namespaces exist on the system?

```sh
$ k get namespaces
```

### How many pods exist in the `research` namespace?

```sh
$ k get pods -n research
```

### Create a POD in the `finance` namespace.

```sh
$ k run redis --image=redis -n finance
```

### Which namespace has the `blue` pod in it?

```sh
$ k get pods --all-namespace
```

### What DNS name should the `Blue` application use to access the database `db-service` in its own namespace - `marketing`?

```sh
$ k get services
```

Host Name: db-service
Host Port: 6379


### What DNS name should the `Blue` application use to access the database `db-service` in the `dev` namespace?

`db-service.dev.svc.cluster.local`
`서비스이름.네임스페이스.svc.cluster.local`


## Practice 5 - Imperative Commands

```sh
$ kubectl apply -f
```

위 명령어를 사용하여 yaml 파일 업데이트 후 적용을 시켜야합니다.

### Deploy a pod named `nginx-pod` using the `nginx:alpine` image.

```sh
$ k run nginx-pod --image=nginx:alpine
```

### Deploy a `redis` pod using the `redis:alpine` image with the labels set to `tier=db`.

```sh
$ k run redis --image=redis:alpine --labels tier=db
```

### Create a service `redis-service` to expose the `redis` application within the cluster on port `6379`.

```sh
$ k expose pod redis --port=6379 --name redis-service --type=ClusterIP
```

### Create a deployment named `webapp` using the image `kodekloud/webapp-color` with `3` replicas.


```sh
$ k create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

### Create a new pod called `custom-nginx` using the `nginx` image and run it on `container port 8080`.

```sh
$ k run custom-nginx --image=nginx --port=8080
```

### Create a new namespace called `dev-ns`.

```sh
$ k create namespace dev-ns
```

### Create a new deployment called `redis-deploy` in the `dev-ns` namespace with the `redis` image. It should have `2` replicas.

```sh
$ k create deploy redis-deploy --image=redis --replicas=2 -n dev-ns
```

### Create a pod called `httpd` using the image `httpd:alpine` in the default namespace. Next, create a service of type `ClusterIP` by the same name `(httpd)`. The target port for the service should be `80`.

```sh
$ k run httpd --image=httpd:alpine --port=80 --expose
```


## Practice 6 - Docker Images

```Dockerfile
$ cat Dockerfile 
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

### Run an instance of the image `webapp-color` and publish port `8080` on the container to `8282` on the host.

```sh
$ docker run -p 8282:8080 webapp-color
```

### What is OS of python:3.6

```sh
$ docker run python:3.6 cat /etc/*release*
```

## Practice 7 - Command and Arguments

### Create a pod using the file named `ubuntu-sleeper-3.yaml`. There is something wrong with it. Try to fix it!

```yml
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-3 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "1200"
```

### Inspect the two files under directory `webapp-color-2`. What command is run at container startup?

```
controlplane ~/webapp-color-2 ➜  cat Dockerfile 
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

controlplane ~/webapp-color-2 ➜  cat webapp-color-pod.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```

```sh
--color green
```
이 실행된디.

## Practice 8 - ConfigMaps

### How many `ConfigMaps` exists in the `default` namespace?

```bash
$ k get configmaps
$ k get cm
```

### Identify the database host from the config map `db-config`.

```bash
$ k describe cm db-config
```

### Create a new ConfigMap for the `webapp-color` POD. Use the spec given below.

```yaml
apiVersion: v1
data:
  APP_COLOR: darkblue
  APP_OTHER: disregard
kind: ConfigMap
metadata:
  name: webapp-config-map
```
### Update the environment variable on the POD to use only the `APP_COLOR` key from the newly created ConfigMap.

#### as-is

```yaml
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
```

#### to-be

```yaml
spec:
  containers:
  - env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map
          key: APP_COLOR
```

## Practice 9 - Secrets

### How many `Secrets` exist on the system?

```bash
$ k get secrets
```

### The reason the application is failed is because we have not created the secrets yet. Create a new secret named `db-secret` with the data given below.


```sh
controlplane ~ ➜  k get all
NAME             READY   STATUS    RESTARTS   AGE
pod/webapp-pod   1/1     Running   0          4m4s
pod/mysql        1/1     Running   0          4m4s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP          15m
service/webapp-service   NodePort    10.43.210.107   <none>        8080:30080/TCP   4m4s
service/sql01            ClusterIP   10.43.129.185   <none>        3306/TCP         4m3s
```

```sh
controlplane ~ ➜  echo sql01 | base64
c3FsMDEK

controlplane ~ ➜  echo root | base64
cm9vdAo=

controlplane ~ ➜  echo password123 | base64
cGFzc3dvcmQxMjMK
```

인코딩이 이상함. 개행 문자가 포함되는 듯?

```yaml
apiVersion: v1
data:
  DB_Host: c3FsMDE=
  DB_User: cm9vdA==
  DB_Password: cGFzc3dvcmQxMjM=
kind: Secret
metadata:
  name: db-secret
  namespace: default
```

```sh
$ k create -f mysql-secret.yaml
```

### Set secret to pod 

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


## Practice 10 - Security Contexts

### What is the user used to execute the sleep process within the `ubuntu-sleeper` pod?

```sh
$ k exec ubuntu-sleeper -- whoami
```

### Run as 1010

```
spec:
  securityContext:
    runAsUser: 1010
```

### A Pod definition file named `multi-pod.yaml` is given. With what user are the processes in the `web` container started?

web: 1002, sidecar: 1001

```yaml
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

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
```

### Update pod `ubuntu-sleeper` to run as Root user and with the `SYS_TIME` capability.


```diff
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-pd27p
      readOnly: true
    securityContext:
+     capabilities:
+       add: ["SYS_TIME"]
```

### Now update the pod to also make use of the `NET_ADMIN` capability.

```diff
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    securityContext:
      capabilities:
        add:
        - SYS_TIME
+       - NET_ADMIN
```

## Practice 11 - Service Account

### How many Service Accounts exist in the default namespace?

```sh
$ k get sa
```

### What is the secret token used by the default service account?

```sh
$ k describe sa default
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
```

### What is the state of the dashboard? Have the pod details loaded successfully?

pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"


### What type of account does the Dashboard application use to query the Kubernetes API?

Service Account

### Which account does the Dashboard application use to query the Kubernetes API?

Default 

```
pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
```

### Inspect the Dashboard Application POD and identify the Service Account mounted on it.

```sh
$ k get po -o yaml
```

```yaml
...
    serviceAccount: default
    serviceAccountName: default
...
```


### At what location is the ServiceAccount credentials available within the pod?

/var/run/secrets

```
Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x458j (ro)
```

### The application needs a ServiceAccount with the Right permissions to be created to authenticate to Kubernetes. The `default` ServiceAccount has limited access. Create a new ServiceAccount named `dashboard-sa`.


```sh
$ k create sa dashboard-sa
```

```sh
controlplane ~ ➜  cat /var/rbac/pod-reader-role.yaml 
---
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

```sh
controlplane ~ ✖ cat /var/rbac/dashboard-sa-role-binding.yaml 
---
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

RBAC 

**RBAC**(Role-Based Access Control, 역할 기반 접근 제어)는 **클러스터 내의 자원에 대한 접근 권한을 제어**하는 메커니즘입니다. RBAC는 사용자가 클러스터의 리소스(예: Pod, 서비스, 네임스페이스 등)에 어떤 작업을 할 수 있는지 정의하고, **역할(role)**을 기반으로 권한을 부여하는 방식으로 동작


### Enter the access token in the UI of the dashboard application. Click `Load Dashboard` button to load Dashboard

```bash
$ kubectl create token dashboard-sa
```

![](Pasted%20image%2020240928160432.png)

### Edit the deployment to change ServiceAccount from `default` to `dashboard-sa`.


```yaml
spec:
	template:
		spec:
	      serviceAccountName: dashboard-sa
```


## Practice 12 - Resource Requirements


### CPU resouce = 1

```yaml
spec:
  containers:
  - args:
	resources:
      limits:
        cpu: "2"
      requests:
        cpu: "1"
```

### Another pod called `elephant` has been deployed in the default namespace. It fails to get to a running state. Inspect this pod and identify the `Reason` why it is not running.

```
$ kubectl describe pod elephant | grep -A5 State:
```

### The `elephant` pod runs a process that consumes 15Mi of memory. Increase the limit of the `elephant` pod to 20Mi.


```yaml
spec:
  containers:
  - args:
    - --vm
    - "1"
    - --vm-bytes
    - 15M
    - --vm-hang
    - "1"
    resources:
      limits:
        memory: 20Mi
      requests:
        memory: 15Mi
```

## Practice 13 - Taints and Tolerations

### How many `nodes` exist on the system?

```
controlplane ~ ➜  k get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   6m4s    v1.30.0
node01         Ready    <none>          5m19s   v1.30.0
```


### has Taints in node01?

```sh
controlplane ~ ➜  k describe node node01 | grep -i taints
Taints:             <none>
```


### Create a taint on `node01` with key of `spray`, value of `mortein` and effect of `NoSchedule`

```sh
$ k taint nodes node01 spray=mortein:NoSchedule
```

### Create a new pod with the `nginx` image and pod name as `mosquito`.

```sh
$ k run --image=nginx mosquito
```

### Create another pod named `bee` with the `nginx` image, which has a toleration set to the taint `mortein`.

```sh
$ k run --image=nginx bee -o yaml > bee.yaml
```

```
spec:
	tolerations:
	  - effect: NoSchedule
	    key: spray
	    value: mortein
```

### Has taints in controlplane node

```sh
controlplane ~ ➜  k describe node controlplane | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

### Remove the taint on `controlplane`, which currently has the taint effect of `NoSchedule`.

```diff
spec:
  podCIDR: 10.244.0.0/24
  podCIDRs:
  - 10.244.0.0/24
- taints:
- - effect: NoSchedule
-   key: node-role.kubernetes.io/control-plane
```


### What is the state of the pod `mosquito` now? Running

### Which node is the POD `mosquito` on now?

```
controlplane ~ ➜  k describe po mosquito | grep -i node
Node:             controlplane/192.10.76.9
```


## Practice 14 - Affinity

### How many label in node01

```sh
controlplane ~ ➜  k describe node node01 | grep -iA5 label
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"12:d0:38:fa:73:6f"}
```

### What is the value set to the label key `beta.kubernetes.io/arch` on `node01`?

amd64
###  Apply a label `color=blue` to node `node01`

```
$ k edit node node01
```

### Create a new deployment named `blue` with the `nginx` image and 3 replicas.

```sh
controlplane ~ ➜  k create deployment blue --image=nginx --replicas=3
deployment.apps/blue created
```

### Which nodes `can` the pods for the `blue` deployment be placed on?

label 방식은 쫒아내지는 않는다.
### Set Node Affinity to the deployment to place the pods on `node01` only.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
```


### Which nodes are the pods placed on now?

```sh
$ kubectl get pods -o wide
```

### Create a new deployment named `red` with the `nginx` image and `2` replicas, and ensure it gets placed on the `controlplane` node only.

Use the label key - `node-role.kubernetes.io/control-plane` - which is already set on the `controlplane` node.

```diff
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: red
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
+     affinity:
+       nodeAffinity:
+         requiredDuringSchedulingIgnoredDuringExecution:
+           nodeSelectorTerms:
+           - matchExpressions:
+             - key: node-role.kubernetes.io/control-plane
+               operator: Exists
```


## Practice 15 - Muliti-Container PODs

### Create a multi-container pod with `2` containers.

Use the spec given below:    
If the pod goes into the `crashloopbackoff` then add the command `sleep 1000` in the `lemon` container.

```sh
$ k run yellow --image=busybox
```

```diff
spec:
  containers:
  - image: busybox
    imagePullPolicy: Always
    name: lemon
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-nt5zk
      readOnly: true
+   command: ["sleep", "1000"]
+ - image: redis
+   name: gold
```

### Inspect the `app` pod and identify the number of containers in it.

It is deployed in the `elastic-stack` namespace.

We will configure a sidecar container for the application to send logs to Elastic Search.

![](Pasted%20image%2020240929160554.png)

```sh
$ k get pods -n elastic-stack
NAME             READY   STATUS    RESTARTS   AGE
app              1/1     Running   0          11m
elastic-search   1/1     Running   0          11m
kibana           1/1     Running   0          11m
```

### The application outputs logs to the file `/log/app.log`. View the logs and try to identify the user having issues with Login.

```sh
$ k exec app -- cat /log/app.log

WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
```

### Edit the pod in the `elastic-stack` namespace to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container.

```sh
$ k get pods app -n elastic-stack -o yaml > app.yaml
```

```diff
spec:
	containers:
+	  - image: kodekloud/filebeat-configured
+	    name: sidecar
+	    volumeMounts:
+	    - mountPath: /var/log/event-simulator/
+	      name: log-volume
```

## Practice 16 - Readiness Probes

```sh
controlplane ~ ➜  k describe po simple-webapp-2 | grep -iA5 port
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 29 Sep 2024 07:19:05 +0000
    Ready:          True
    Restart Count:  0
    Environment:

controlplane ~ ➜  k describe po simple-webapp-1 | grep -iA5 port
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 29 Sep 2024 07:17:36 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
```

```sh
controlplane ~ ➜  cat curl-test.sh 
for i in {1..20}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http://webapp-service.default.svc.cluster.local:8080/ready 2>&1` && echo "$test OK" || echo "Failed"';
   echo ""
done
```

### Update the newly created pod `'simple-webapp-2'` with a readinessProbe using the given spec

```diff
spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
+   readinessProbe:
+     httpGet:
+       path: /ready
+       port: 8080
```

### What would happen if the application inside container on one of the PODs crashes?

```sh
cat crash-app.sh 
kubectl exec --namespace=kube-public curl -- wget -qO- http://webapp-service.default.svc.cluster.local:8080/crash
```


죽이면? restart

### What would happen if the application inside container on one of the PODs freezes?

```sh
controlplane ~ ➜  cat freeze-app.sh 
nohup kubectl exec --namespace=kube-public curl -- wget -qO- http://webapp-service.default.svc.cluster.local:8080/freeze &
```

새로운 유저들에게 영향을 미친다 (?)
그냥 전체 유저에 대해서 여향을 미치는게 아닌가? 

### Update both the pods with a livenessProbe using the given spec

```diff
spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
+   livenessProbe:
+     httpGet:
+       path: /live
+       port: 8080
+     periodSeconds: 1
+     initialDelaySeconds: 80
```

=> 해설 봐야 함.

## Practice 17 - Logging

### A user - `USER5` - has expressed concerns accessing the application. Identify the cause of the issue.

```sh
$ k logs -f pods/webapp-1
```

```
[2024-09-29 07:56:19,457] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
```


## Practice 18 - Monitoring

```sh
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
```

### Deploy the metrics-server by creating all the components downloaded.

```sh
$ cd kubernetes-metrics-server
$ k create -f .
```

```sh
controlplane ~ ➜  k top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   243m         0%     1139Mi          0%        
node01         23m          0%     280Mi           0%
```

### Identify the node that consumes the `most` CPU(cores).

```sh
controlplane ~ ➜  k top node --sort-by='cpu'
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   235m         0%     1145Mi          0%        
node01         23m          0%     283Mi           0%        
```

```sh
controlplane ~ ➜  k top node --sort-by='memory'
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   238m         0%     1146Mi          0%        
node01         28m          0%     282Mi           0%
```

## Practice 19 - Init Containers

### Identify the pod that has an `initContainer` configured.

```sh
$ k describe pods | grep -iA5 "init Container"
```


### We just created a new app named `purple`. How many `initContainers` does it have?

```sh
 $ k get po purple -o yaml
```

```yaml
initContainers:
  - command:
    - sh
    - -c
    - sleep 600
    image: busybox:1.28
    imagePullPolicy: IfNotPresent
    name: warm-up-1
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-tblmr
      readOnly: true
  - command:
    - sh
    - -c
    - sleep 1200
    image: busybox:1.28
    imagePullPolicy: IfNotPresent
    name: warm-up-2
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-tblmr
      readOnly: true
```

### Update the pod `red` to use an `initContainer` that uses the `busybox` image and `sleeps for 20` seconds

```diff
spec:
  initContainers:
+ - command:
+   - sleep
+   - "20"
+   name: red-initcontainer
+   image: busybox
```

### A new application `orange` is deployed. There is something wrong with it. Identify and fix the issue.


```diff
  initContainers:
  - command:
    - sh
    - -c
-   - sleeeep 2;
+   - sleep 2;
```

## Practice 20 - Labels and Selectors

We have deployed a number of PODs. They are labelled with `tier`, `env` and `bu`. How many PODs exist in the `dev` environment (`env`)?

```sh
$ k get pods -l env=dev
```

```sh
k get pods --selector bu=finance
```

### How many objects are in the `prod` environment including PODs, ReplicaSets and any other objects?

```sh
$ k get all --selector env=prod
```

### Identify the POD which is part of the `prod` environment, the `finance` BU and of `frontend` tier?

```sh
$ k get pods --selector env=prod,bu=finance,tier=frontend
```

### A ReplicaSet definition file is given `replicaset-definition-1.yaml`. Attempt to create the replicaset; you will encounter an issue with the file. Try to fix it.

```diff
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
-       tier: nginx
+       tier: front-end
     spec:
       containers:
       - name: nginx
         image: nginx
```

## Practice 21 - Rolling Updates & Rollbacks

###  What is the current color of the web application?

blue 

### Up to how many PODs can be down for upgrade at a time

RollingUpdateStrategy:  25% max unavailable, 25% max surge

### Change the deployment strategy to `Recreate`

```yaml
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

```
  strategy:
    type: Recreate
```

## Practice 22 - Jobs and CronJobs

### Create a Job using this POD definition file or from the imperative command and look at how many attempts does it take to get a `'6'`.

```sh
$ create job throw-dice-job --image=kodekloud/throw-dice --dry-run=client -o yaml > throw-dice-job.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: throw-dice-job
spec:
  backoffLimit: 15 # This is so the job does not quit before it succeeds.
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
status: {}
```

```sh
controlplane ~ ➜  k get pods
NAME                   READY   STATUS      RESTARTS   AGE
throw-dice-job-bg9kk   0/1     Error       0          2m1s
throw-dice-job-gqh9f   0/1     Error       0          110s
throw-dice-job-q5pmh   0/1     Error       0          89s
throw-dice-job-z5wb4   0/1     Completed   0          48s
throw-dice-pod         0/1     Error       0          6m54s

controlplane ~ ➜  k logs -f throw-dice-job-z5wb4
6
```
### Update the job definition to run as many times as required to get `2` successful `6's`.

```diff 
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: throw-dice-job
spec:
+ completions: 2
+ backoffLimit: 25
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
status: {}
```

### That took a while. Let us try to speed it up, by running upto `3` jobs in parallel.

```diff
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: throw-dice-job
spec:
- completions: 2
+ completions: 3
+ parallelism: 3
  backoffLimit: 25
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice-job
        resources: {}
      restartPolicy: Never
status: {}
```

### cronjob

```sh
$ k create cronjob throw-dice-cron-job --schedule="30 21 * * *" --image=kodekloud/throw-dice --dry-run=client -o yaml > throw-dice-cron-job.yaml
```

## Practice 23 - Kubernetes Services

```sh
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   24m
```
That is a default service created by Kubernetes at launch.

### What is the type of the default `kubernetes` service?

ClusterIP

### What is the `targetPort` configured on the `kubernetes` service?

```sh
controlplane ~ ➜  k describe service kubernetes 
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.0.1
IPs:               10.43.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         192.2.153.3:6443
Session Affinity:  None
Events:            <none>
```

### How many labels are configured on the `kubernetes` service?

```
Labels:            component=apiserver
                   provider=kubernetes
```

### How many Endpoints are attached on the `kubernetes` service?

```
Endpoints:         192.2.153.3:6443
```

### How many Deployments exist on the system now?

```sh
controlplane ~ ➜  k get deploy
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
simple-webapp-deployment   4/4     4            4           17s
```

### What is the image used to create the pods in the deployment?

```
Pod Template:
  Labels:  name=simple-webapp
  Containers:
   simple-webapp:
    Image:         kodekloud/simple-webapp:red
```

### Create a new service to access the web application using the `service-definition-1.yaml` file.

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports:
  - nodePort:  30080
    port: 8080 
    targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort
```

## Practice 24 - Network Policies

### How many network policies do you see in the environment?

```sh
controlplane ~ ✖ k get networkpolicies
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   50s
```

### Which pod is the Network Policy applied on?

```sh
$ k get netpol
$ k get pods --show-labels | grep name=payroll
payroll    1/1     Running   0          2m56s   name=payroll
```

### What type of traffic is this Network Policy configured to handle?

```

```

## Practice 25 - Ingress Networking - 1

## Practice 26 - Ingress Networking - 2
