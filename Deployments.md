
![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (8).png]]

![[www.udemy.com_course_certified-kubernetes-application-developer_learn_lecture_12316794 (9).png]]


```
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels: 
		app: myapp
		type: front-end

spec:
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
				- name: nginx-container
				  image: nginx

	replicas: 3
	selector:
		matchLabels:
			type: front-end
```

```
$ kubectl get all
$ kubectl get deploy
$ kubectl get deployment
```


```yml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: httpd-frontend
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: httpd-pod
      labels:
        app: httpd
        type: front-end
    spec:
      containers:
        - name: httpd-container
          image: httpd:2.4-alpine
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```
$ kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```




```
$ kubectl create namespace test-123 --dry-run -o json
```

위와 같이 명령어에 대한 결과물을 json 형태로 출력되록 설정할 ㅅ수 있다.