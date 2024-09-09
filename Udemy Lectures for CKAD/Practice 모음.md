
## Section 2

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

## Section 3

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

### Now scale the ReplicaSet down to 2 PODs.

