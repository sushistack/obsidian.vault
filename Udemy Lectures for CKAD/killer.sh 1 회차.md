### Question 1 | Contexts


You have access to multiple clusters from your main terminal through kubectl contexts. Write all those context names into /opt/course/1/contexts.

```sh
$ k config get-contexts -o name > /opt/course/1/contexts
```

Next write a command to display the current context into /opt/course/1/context_default_kubectl.sh, the command should use kubectl.

```sh
$ echo 'kubectl config current-context' > /opt/course/1/context_default_kubectl.sh
```

Finally write a second command doing the same thing into /opt/course/1/context_default_no_kubectl.sh, but without the use of kubectl.

```sh
$ cat ~/.kube/config | grep current-context | sed -e "s/current-context: //"
```

---

### Question 2 | Schedule Pod on Controlplane Nodes

Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container. This Pod should only be scheduled on controlplane nodes. Do not add new labels to any nodes.

```sh
$ k run pod1 --image=httpd:2.4.41-alpine --dry-run=client -o yaml > pod1.yml
```

```
tolerations:
	- key: xx
	  operator: "Exists"
	  effect: xxx
```

---

### Question 3 | Scale down StatefulSet

There are two Pods named o3db-* in Namespace project-c13. C13 management asked you to scale the Pods down to one replica to save resources.

```sh
$ k scale sts o3db -n project-c13  --replicas=1
```
`

---

### Question 4 | Pod Ready if Service is reachable

Do the following in Namespace default. Create a single Pod named ready-if-service-ready of image nginx:1.16.1-alpine. Configure a LivenessProbe which simply executes command true. Also configure a ReadinessProbe which does check if the url [http://service-am-i-ready:80](http://service-am-i-ready/) is reachable, you can use wget -T2 -O- [http://service-am-i-ready:80](http://service-am-i-ready/) for this. Start the Pod and confirm it isn't ready because of the ReadinessProbe.

```sh
$ k run pod4 --image=nginx:1.16.1-alpine --dry-run=client -o yaml > pod4.yml
```

```
apiVersion: v1  
kind: Pod  
metadata:  
  labels:  
    run: pod4  
  name: pod4  
spec:  
  containers:  
    - image: nginx:1.16.1-alpine  
      name: pod4  
      resources: {}  
      readinessProbe:  
        exec:  
          command:  
            - sh  
            - '-c'  
            - 'wget -T2 -O- http://service-am-i-ready:80'  
      livenessProbe:  
        exec:  
          command:  
            - 'true'  
  dnsPolicy: ClusterFirst  
  restartPolicy: Always
```

Create a second Pod named am-i-ready of image nginx:1.16.1-alpine with label id: cross-server-ready. The already existing Service service-am-i-ready should now have that second Pod as endpoint.

```sh
$ k run am-i-ready --image=nginx:1.16.1-alpine --dry-run=client -o yaml > am-i-ready.yml
```



Now the first Pod should be in ready state, confirm that.

---

### Question 5 | Kubectl sorting

There are various Pods in all namespaces. Write a command into /opt/course/5/find_pods.sh which lists all Pods sorted by their AGE (metadata.creationTimestamp).

```sh
$ echo 'kubectl get pods --all-namespaces --sort-by=metadata.creationTimestamp' > /opt/course/5/find_pods.sh
```

Write a second command into /opt/course/5/find_pods_uid.sh which lists all Pods sorted by field metadata.uid. Use kubectl sorting for both commands.

```sh
$ echo 'kubectl get pods --all-namespaces --sort-by=metadata.uid' > /opt/course/5/find_pods.sh
```

---

### Question 6 | Storage, PV, PVC, Pod volume

Create a new PersistentVolume named safari-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: safari-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 2Gi
  hostPath:
    path: /Volumes/Data
```

Next create a new PersistentVolumeClaim in Namespace project-tiger named safari-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: safari-pvc
  namespace: project-tiger

spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

Finally create a new Deployment safari in Namespace project-tiger which mounts that volume at /tmp/safari-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

```sh
$ kubectl create deploy safari -n project-tiger --image=httpd:2.4.41-alpine -o yaml > safari-deploy.yml
```

---

### Question 7 | Node and Pod Resource Usage

The metrics-server has been installed in the cluster. Your college would like to know the kubectl commands to:

1. show Nodes resource usage -> `kubectl top node`
2. show Pods and their containers resource usage -> `kubectl top pod --containers=true`

Please write the commands into /opt/course/7/node.sh and /opt/course/7/pod.sh.

---

### Question 8 | Get Controlplane Information

Ssh into the controlplane node with ssh cluster1-controlplane1. Check how the controlplane components kubelet, kube-apiserver, kube-scheduler, kube-controller-manager and etcd are started/installed on the controlplane node. Also find out the name of the DNS application and how it's started/installed on the controlplane node.


Write your findings into file /opt/course/8/controlplane-components.txt. The file should be structured like:

> /opt/course/8/controlplane-components.txt  
> kubelet: [TYPE]  
> kube-apiserver: [TYPE]  
> kube-scheduler: [TYPE]  
> kube-controller-manager: [TYPE]  
> etcd: [TYPE]  
> dns: [TYPE][NAME]

Choices of [TYPE] are: not-installed, process, static-pod, pod

> kubelet: process  
> kube-apiserver: static-pod  
> kube-scheduler: static-pod  
> kube-controller-manager: static-pod  
> etcd: static-pod  
> dns: pod coredns

---

### Question 9 | Kill Scheduler, Manual Scheduling

Ssh into the controlplane node with ssh cluster2-controlplane1. Temporarily stop the kube-scheduler, this means in a way that you can start it again afterwards.

Create a single Pod named manual-schedule of image httpd:2.4-alpine, confirm it's created but not scheduled on any node.

Now you're the scheduler and have all its power, manually schedule that Pod on node cluster2-controlplane1. Make sure it's running.

Start the kube-scheduler again and confirm it's running correctly by creating a second Pod named manual-schedule2 of image httpd:2.4-alpine and check if it's running on cluster2-node1.

---

### Question 10 | RBAC ServiceAccount Role RoleBinding

Create a new ServiceAccount processor in Namespace project-hamster. Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

---

### Question 11 | DaemonSet on all Nodes

Use Namespace project-tiger for the following. Create a DaemonSet named ds-important with image httpd:2.4-alpine and labels id=ds-important and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The Pods it creates should request 10 millicore cpu and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, also controlplanes.

---

### Question 12 | Deployment on all Nodes

Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image google/pause.

There should be only ever one Pod of that Deployment running on one worker node. We have two worker nodes: cluster1-node1 and cluster1-node2. Because the Deployment has three replicas the result should be that on both nodes one Pod is running. The third Pod won't be scheduled, unless a new worker node will be added.

In a way we kind of simulate the behaviour of a DaemonSet here, but using a Deployment and a fixed number of replicas.

---

### Question 13 | Multi Containers and Pod shared Volume

Create a Pod named multi-container-playground in Namespace default with three containers, named c1, c2 and c3. There should be a volume attached to that Pod and mounted into every container, but the volume shouldn't be persisted or shared with other Pods.

Container c1 should be of image nginx:1.17.6-alpine and have the name of the node where its Pod is running available as environment variable MY_NODE_NAME.

Container c2 should be of image busybox:1.31.1 and write the output of the date command every second in the shared volume into file date.log. You can use while true; do date >> /your/vol/path/date.log; sleep 1; done for this.

Container c3 should be of image busybox:1.31.1 and constantly send the content of file date.log from the shared volume to stdout. You can use tail -f /your/vol/path/date.log for this.

---

### Question 14 | Find out Cluster Information

You're ask to find out following information about the cluster k8s-c1-H:

1. How many controlplane nodes are available?
2. How many worker nodes are available?
3. What is the Service CIDR?
4. Which Networking (or CNI Plugin) is configured and where is its config file?
5. Which suffix will static pods have that run on cluster1-node1?

---

### Question 15 | Cluster Event Logging

Write a command into /opt/course/15/cluster_events.sh which shows the latest events in the whole cluster, ordered by time (metadata.creationTimestamp). Use kubectl for it.

Now delete the kube-proxy Pod running on node cluster2-node1 and write the events this caused into /opt/course/15/pod_kill.log.

Finally kill the containerd container of the kube-proxy Pod on node cluster2-node1 and write the events into /opt/course/15/container_kill.log.

Do you notice differences in the events both actions caused?

---

### Question 18 | Fix Kubelet

Use context: kubectl config use-context k8s-c3-CCC

There seems to be an issue with the kubelet not running on cluster3-node1. Fix it and confirm that cluster has node cluster3-node1 available in Ready state afterwards. You should be able to schedule a Pod on cluster3-node1 afterwards.

Write the reason of the issue into /opt/course/18/reason.txt.


---

### Question 20 | Update Kubernetes Version and join cluster

Your coworker said node cluster3-node2 is running an older Kubernetes version and is not even part of the cluster. Update Kubernetes on that node to the exact version that's running on cluster3-controlplane1. Then add this node to the cluster. Use kubeadm for this.

---

### Question 24 | NetworkPolicy

There was a security incident where an intruder was able to access the whole cluster from a single hacked backend Pod.

To prevent this create a NetworkPolicy called np-backend in Namespace project-snake. It should allow the backend-* Pods only to:

- connect to db1-* Pods on port 1111
- connect to db2-* Pods on port 2222
    

Use the app label of Pods in your policy.

After implementation, connections from backend- _Pods to vault-_ Pods on port 3333 should for example no longer work.

---

### Question 25 | Etcd Snapshot Save and Restore

Make a backup of etcd running on cluster3-controlplane1 and save it on the controlplane node at /tmp/etcd-backup.db.

Then create any kind of Pod in the cluster.

Finally restore the backup, confirm the cluster is still working and that the created Pod is no longer with us.
