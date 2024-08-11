
```
$ kubectl create deployment --image=nginx nginx
```

```
$ kubectl create deployment --image=nginx nginx --dry-run -o yaml
$ kubectl create deployment nginx --image=nginx --replicas=4
$ kubectl scale deployment nginx --replicas=4

$ kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

```
$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
$ kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
$ kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```


### Practice



```
$ kubectl run nginx-pod --image=nginx:alpine
```


```
$ kubectl run redis -l tier=db --image=redis:alpine
```

or

```yml
apiVersion: v1
kind: Pod
metadata:
	labels:
		tier: db 
	name: redis 
spec: 
	containers:
		- image: redis:alpine
		  name: redis
	dnsPolicy: ClusterFirst
	restartPolicy: Always
```


Create a service `redis-service` to expose the `redis` application within the cluster on port `6379`

```
$ k expose pod redis --port=6379 --name redis-service
```

Create a deployment named `webapp` using the image `kodekloud/webapp-color` with `3` replicas.

```
$ k create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

Create a new pod called `custom-nginx` using the `nginx` image and run it on `container port 8080`.

```
$ k run custom-nginx --image=nginx --port=8080
```

```
Containers:
  custom-nginx:
    Container ID:   containerd://3a7e5b35cdba4282a11f23d797df96373027c4c7bc92b78866a471e0c438268e
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:6af79ae5de407283dcea8b00d5c37ace95441fd58a8b1d2aa1ed93f5511bb18c
    Port:           8080/TCP
```