### PV / PVC 

Create a Persistent Volume called `log-volume`. It should make use of a `storage class` name `manual`. It should use `RWX` as the access mode and have a size of `1Gi`. The volume should use the hostPath `/opt/volume/nginx`  

Next, create a PVC called `log-claim` requesting a minimum of `200Mi` of storage. This PVC should bind to `log-volume`.  

Mount this in a pod called `logger` at the location `/var/www/nginx`. This pod should use the image `nginx:alpine`.




```
spec:
  containers:
  - image: kodekloud/nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /
        port: 9080
        scheme: HTTP
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
    resources: {}
```


```sh
$ k create cronjob my-job --image=kodekloud/throw-dice --schedule="*/1 * * * *" --dry-run=client -o yaml > dice-cronjob.yml
```

```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dice
spec:
  jobTemplate:
    metadata:
      name: dice
    spec:
      backoffLimit: 25
      ttlSecondsAfterFinished: 20
      template:
        metadata:
        spec:
          containers:
          - image: kodekloud/throw-dice
            name: my-job
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
status: {}
```

```
k run my-busybox --image=busybox -n dev2406 --dry-run=client -o yaml > busybox.yml
```

```
 k describe node controlplane | grep -iA5 label
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=controlplane
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
```
