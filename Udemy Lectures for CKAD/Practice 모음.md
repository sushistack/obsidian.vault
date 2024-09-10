
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


### Practice 5 - Imperative Commands

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




## Practice 7 - ConfigMaps

## Practice 8 - Secrets