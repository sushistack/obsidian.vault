각각의 Kubernetes 리소스에 대한 간단한 예제 YAML 파일을 보여드리겠습니다. 이 예제들은 각 리소스의 주요 기능을 간단하게 나타내는 것입니다.

---

### 1. **Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    env:
    # ConfigMap을 환경 변수로 설정
    - name: CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: example-configmap
          key: config-key
    - name: APP_NAME
      valueFrom:
        configMapKeyRef:
          name: example-configmap
          key: app-name

    # Secret을 환경 변수로 설정
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: example-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: example-secret
          key: password
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    volumeMounts:
    # ConfigMap을 /etc/config 경로에 파일로 마운트
    - name: config-volume
      mountPath: /etc/config
    # Secret을 /etc/secret 경로에 파일로 마운트
    - name: secret-volume
      mountPath: /etc/secret
  volumes:
  # ConfigMap을 볼륨으로 사용
  - name: config-volume
    configMap:
      name: example-configmap
  # Secret을 볼륨으로 사용
  - name: secret-volume
    secret:
      secretName: example-secret
```

```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    envFrom:
    # ConfigMap을 환경 변수로 가져오기
    - configMapRef:
        name: example-configmap
    
    # Secret을 환경 변수로 가져오기
    - secretRef:
        name: example-secret
```


---

### 2. **ReplicaSet**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

---

### 3. **Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

---

### 4. **StatefulSet**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

---

### 5. **DaemonSet**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
spec:
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

---

### 6. **ConfigMap**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  example.key: "example-value"
```

---

### 7. **Secret**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=  # base64 인코딩된 값
```

---

### 8. **ServiceAccount**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: example-serviceaccount
```

---

### 9. **Role**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: example-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

---

### 10. **RoleBinding**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: example-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: example-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: example-role
  apiGroup: rbac.authorization.k8s.io
```

---

### 11. **Job**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["echo", "Hello, World!"]
      restartPolicy: Never
```

---

### 12. **CronJob**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox
            command: ["echo", "Hello from CronJob!"]
          restartPolicy: Never
```

---

### 13. **NetworkPolicy**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-networkpolicy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

---

### 14. **Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

---

### 15. **PersistentVolume (PV)**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

---

### 16. **PersistentVolumeClaim (PVC)**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

### 17. **StorageClass**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-storageclass
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

---

이 예제들은 각 Kubernetes 리소스의 기본적인 사용 방법을 보여줍니다.