![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (10).png]]

![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (11).png]]

![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (12).png]]

![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (13).png]]

![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (14).png]]

```
mysql.connect("db-service.dev.svc.cluster.local")
```


```
$ kubectl get pods --namespace=kube-system
$ kubectl create -f pod-definition.yml --namespace=dev
```

```yml
apiVersion: v1
kind: Namespace
metdadata:
	name: dev
```

```
$ kubectl create -f namespace-dev.yml
```

or

```
$ kubectl create namespace dev
```

Switch

```
$ kubectl config set-context $(kubectl config current-context) --namespace=dev
```


![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (15).png]]



```
$ kubectl get namespaces
$ kubectl get pods --namespace=research
$ kubectl run redis --image=redis --namespace=finance
$ kubectl get svc -n=dev
```


---


```
$ kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

기본적으로 명령을 실행하는 즉시 리소스가 생성됩니다. 단순히 명령을 테스트하려면 --dry-run=client 옵션을 사용하십시오. 그러면 리소스가 생성되지 않습니다. 대신 리소스를 생성할 수 있는지 여부와 명령이 올바른지 여부를 알려주십시오.

