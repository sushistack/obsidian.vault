## PV / PVC 관련 문제

### Create a Persistent Volume called `log-volume`. It should make use of a `storage class` name `manual`. It should use `RWX` as the access mode and have a size of `1Gi`. The volume should use the hostPath `/opt/volume/nginx`  

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec: 
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  storageClassName: manual
  hostPath:
    path: /opt/volume/nginx
```

### Next, create a PVC called `log-claim` requesting a minimum of `200Mi` of storage. This PVC should bind to `log-volume`.

```diff
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: log-claim
spec:
+ storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```

PVC 도 같이 맞춰줘야 함.
### Mount this in a pod called `logger` at the location `/var/www/nginx`. This pod should use the image `nginx:alpine`.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: logger
  namespace: default
spec:
  containers:
    - image: nginx:alpine
      name: nginx-container
      resources: {}
      volumeMounts:
      - mountPath: /var/www/nginx
        name: mypvc
  volumes:
    - name: mypvc
      persistentVolumeClaim:
        claimName: log-claim
```


## 2. Service 관련 문제

We have deployed a new pod called `secure-pod` and a service called `secure-service`. Incoming or Outgoing connections to this pod are not working.  
Troubleshoot why this is happening.  

Make sure that incoming connection from the pod `webapp-color` are successful.

Important: Don't delete any current objects deployed.

### 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: time-checker
  namespace: dvl1987
data:
  TIME_FREQ: "10"
```

```
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: dvl1987

spec:
  containers:
    - name: time-check-container
      image: busybox
      command: ["while", "true;", "do", "date;", "sleep", "$TIME_FREQ;", "done"]
      env:
      - name: TIME_FREQ
        valueFrom:
          configMapKeyRef:
            name: time-config
            key: TIME_FREQ
```
