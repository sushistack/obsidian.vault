
```yaml
apiVerion: v1

kind: Pod

metadata:
	name: myapp-pod
	labels:
		app: myapp
		# costcenter: amer
		# location: NA

spec:
	containers:
		- name: nginx-container
		  image: nginx
		
		- name: backend-container
		  image: redis
```

```
$ kubectl get pods
$ kubectl create -f pod-definition.yml
```

```
$ kubectl run -f redis.yaml
$ kubectl apply -f redis.yaml
```

허브로 부터 받은 pod에서 definition.yaml 뽑아내기

```
$ kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```

```
kubectl edit pod <pod-name>
```