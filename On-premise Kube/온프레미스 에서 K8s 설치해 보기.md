
## K8s 구성

### Kubeadm 을 이용한 K8s 구축

* host file 및 iptables 정리

```sh
sed -i '/novalocal/d' /etc/hosts
sed -i '/ip6/d' /etc/hosts
update-alternatives --set iptables /usr/sbin/iptables-legacy
```

* 본인의 k8s hostname과 IP를 넣을 것

```sh
cat << EOF >> /etc/hosts
192.168.0.110  btpa-onk8s-ca818
192.168.0.92  btpa-onk8s-cb818
EOF
```

* Sysctl configuration

```sh
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

chattr -i /etc/sysctl.conf
cat << EOT > /etc/sysctl.conf
net.ipv4.ip_forward = 1
EOT
chattr +i /etc/sysctl.conf
sysctl -p 

sysctl --system
```

* swapoff

```sh
swapoff -a
sed -i '/swap/s/^/#/g' /etc/fstab
```

* docker 설치

```sh
apt update

curl -s https://get.docker.com | sh
systemctl enable --now docker.service
```

* kubelet, kubeadm, kubectl 설치 (kubernetes.io docs Page 에서 한글 버전과 engilsh 버전이 다름)

```sh
apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


apt update 

apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

systemctl --now enable kubelet
```

* kubelet 및 docker 환경 설정

```sh
### 다만 아래 설정은 우리 k8s가 cri 로 docker 를 사용하지 않으므로, 필요 없음 
cat << EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

```sh
sed -i '3s/\/kubelet.conf/\/kubelet.conf --cgroup-driver=systemd/g' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

systemctl daemon-reload
systemctl restart docker kubelet containerd systemd-resolved
```

* Master node 설정
    * 설치 이후 나오는 Master/Woker node 추가 값은 24시간이 지나면 Token 이 만료 됨, 따라서 24시간이 지났을 경우 별도 명령어를 수행 하여 node 추가 하여야 함.

```sh
kubeadm init --control-plane-endpoint=`ip a s | grep 192.168. | awk '{print $2}' | cut -d "/" -f1`:6443 --pod-network-cidr=172.16.0.0/16 --upload-certs 
kubeadm init --control-plane-endpoint=자신의_haproxy_IP:6443 --pod-network-cidr=172.16.0.0/16 --upload-certs
```

### 설치 이후 작업

* Worker node 추가 (Master 노드에서 아래 커맨드 입력 후 결과 값을 Worker node 에서 입력)

```sh
kubeadm token create --print-join-command 
```

* Master node 추가 (Master 노드에서 아래 커맨드 입력 후 결과 값을 Worker node 에서 입력)

```sh
kubeadm token create --certificate-key $(kubeadm init phase upload-certs --upload-certs|tail -n 1) --print-join-command
```

* kubeconfig 파일 이동 (Master node 에서 입력)

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

* 자동 완성 및 alias (\~/.bashrc 에 입력)

```sh
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
alias k='/usr/bin/kubectl'
```

* Master 노드와 worker 노드

| 역할 | Master 노드 | Worker 노드 |
| --- | --------- | --------- |
| 주요 역할 | 클러스터의 관리 및 제어 (Control Plane) | 애플리케이션 및 서비스 실행 |
| @rows=4:컴포넌트 | \- kube\-apiserver: API 요청을 처리 | \- kubelet: 파드의 상태를 관리하고 노드에서 파드를 실행 |
| \- kube\-controller\-manager: 클러스터 상태 관리 | \- kube\-proxy: 네트워크 트래픽 라우팅 |
| \- kube\-scheduler: 파드를 적절한 노드에 스케줄링 | \- 실행 중인 파드\(Pod\) |
| \- etcd: 클러스터 데이터를 저장 \(Key\-Value store\) | \- 클러스터의 실제 애플리케이션을 실행하는 노드 |
| 주요 기능 | \- 클러스터 상태 모니터링 | \- 파드 실행 및 관리 |
| @rows=3:서비스 노출 | \- 스케줄링 및 리소스 할당 | \- 서비스 및 애플리케이션의 네트워크 액세스 제공 |
| \- 클러스터 API 제공 및 인증 처리 | \- 파드 상태 체크\, 로그 수집\, 노드 자원 활용 |
| 클러스터 내부에서만 서비스 노출 (일반적으로 외부와 직접 연결 없음) | 서비스를 통해 외부와 연결 가능 (NodePort, LoadBalancer 등) |
| 고가용성 | 클러스터 관리를 위한 노드. 여러 개의 Master 노드를 배치해 고가용성 보장 | 애플리케이션 배포를 위한 노드. 장애 발생 시, 다른 Worker 노드에서 서비스 지속 |
| 에러 발생 시 | Master 노드가 죽으면, 클러스터 상태 변경이나 스케줄링에 영향 있음 | Worker 노드가 죽으면, 실행 중인 파드에 영향. 복제본을 통한 서비스 복구 가능 |

* 재설치

```sh
kubeadm reset
```

- 전체 재설치 스크립트 (calico 까지 설치)
```sh
cat << EOT > ./reinstall_k8s.sh

#!/bin/bash

## 마스터 노드 kubeadm reset
kubeadm reset -f
rm -rf /etc/cni/net.d

## 마스터 노드 재 설치

read -p "본인의 HAProxy IP를 입력하세요:" H_IP
kubeadm init --control-plane-endpoint=\$H_IP:6443 --pod-network-cidr=172.16.0.0/16 --upload-certs

find / -name test2_ysh.pem -exec cp -avfpr {} ./ \;

mkdir -p \$HOME/.kube
sudo cp /etc/kubernetes/admin.conf \$HOME/.kube/config
sudo chown \$(id -u):\$(id -g) \$HOME/.kube/config

WC=\`kubeadm token create --print-join-command\`

## 워커 노드 kubeadm reset 및 재 설치

read -p "본인의 Worker node IP를 입력하세요:" W_IP

echo "워커노드 reset 진행 중..."
ssh -i ./test2_ysh.pem ubuntu@\$W_IP "sudo kubeadm reset -f ; sudo rm -rf /etc/cni/net.d"
ssh -i ./test2_ysh.pem ubuntu@\$W_IP "sudo \$WC"
ssh -i ./test2_ysh.pem ubuntu@\$W_IP "sudo systemctl restart kubelet containerd"

echo "워커노드 재 설치 완료"

## calico 설치
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml -O

sed -i "s/192.168.0.0/172.16.0.0/g" ./custom-resources.yaml

kubectl create -f custom-resources.yaml

## 최종 node 확인
 systemctl restart containerd
 systemctl restart kubelet

kubectl get no


EOT

```


### Haproxy 를 활용한 Master 다중화 설명

```sh
apt update 
apt install haproxy  

cat << EOF > /etc/haproxy/haproxy.cfg
global
   maxconn      4096
   nbproc       2
   log          /dev/log local0
   log          /dev/log local1 notice

defaults
   log global
   timeout http-request    10s
   timeout queue           1m
   timeout connect         10s
   timeout client          1m
   timeout server          1m
   timeout http-keep-alive 10s
   timeout check           10s

frontend kubernetes-master-lb
   bind 0.0.0.0:6443
   option tcplog
   mode tcp
   default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
   mode tcp
   balance roundrobin
   option tcp-check
   option tcplog
   server {마스터서버Hostname} {마스터서버IP}:6443 check
EOF

```
## Calico CNI를 이용한 network 구성

### Calico 구성

* k8s 내부에서의 네트워크 통신.
    * Pod to Pod
        ![](Pasted%20image%2020250613092132.png)
    * 멀티노드 pod
        ![](Pasted%20image%2020250613092228.png)
* CNI(Container Network Interface)란 무엇인가?
    * 컨테이너 간의 네트워킹을 제어할 수 있는 플러그인의 표준.
    * ![](Pasted%20image%2020250616092624.png)
    
	
* 내 cidr  알아 보려면
```sh
cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr
```

* Calico 설치

```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/tigera-operator.yaml

curl https://raw.githubusercontent.com/projectcalico/calico/v3.30.0/manifests/custom-resources.yaml -O

kubectl create -f custom-resources.yaml
```


### Package 관리자 helm

* helm 설치

```sh
mkdir helm
cd helm
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```


## Dynamic ephemeral volume

### PV, PVC, SC 그리고 Provisioner

![](volume_type.gif)

* PV (PersistentVolume)
    * K8s 클러스터 내의 저장소를 추상화한 객체이며 사용자가 데이터를 저장할 수 있는 실제 저장소, 볼륨 그 자체를 뜻함.
* PVC (PersistentVolumeClaim)
    * 사용자가 PV를 요청하기 위해 생성하는 K8s 객체.
    * PVC는 요청된 저장소 크기와 접근 모드를 기반으로 클러스터 내 사용가능한 PV를 검색하여 바인딩.
    * 사용하고 싶은 용량은 얼마인지, 읽기/쓰기는 어떤 모드로 설정하고 싶은지 등을 정해서 요청.
* SC (StorageClass)
    * 동적 PV 프로비저닝을 정의하는 객체
    * PVC가 특정 StorageClass 를 요청하면, 그에 따라 PV가 동적으로 생성 됨.
* Provisioner
    * PV를 동적으로 생성하는 백엔드 구성요소.
    * SC와 연결되어 동작하며, 설정에 따라 적절한 스토리지를 프로비저닝하고, PV를 자동생성함.
* ServiceAccount(SA)
* Role
    * 특정 Namespace 내에서 접근을 제어하는 권한을 정의하는 객체
* RoleBinding
    * Role 을 SA에게 연결하는 객체, Role 이 권한을 실제로 사용할 수 있도록 연결
* ClusterRole
    * Role 과 비슷하지만 Cluster 전체에 적용
* ClusterRoleBiding
    * Cluster Role 을 SA에게 연결하는 객체

### Persistent Volume 설정

* provisioner, sa, pvc 설정

```sh
cat << EOT > provisioner.yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-pod-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-pod-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-pod-provisioner
    spec:
      serviceAccountName: nfs-pod-provisioner-sa # name of service account
      containers:
        - name: nfs-pod-provisioner
          image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-provisioner-volume
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME # do not change
              value: nfs-provisioner # SAME AS PROVISIONER NAME VALUE IN STORAGECLASS
            - name: NFS_SERVER # do not change
              value: 192.168.0.9 ## NAS 서버의 IP설정
            - name: NFS_PATH 
              value: /bpt-nas  # NAS 서버의 세부 Path 설정
      volumes:
       - name: nfs-provisioner-volume # same as volumemouts name
         nfs:
           server: 192.168.0.9
           path: /btp-nas
EOT
```

```sh
cat << EOT > sa.yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-pod-provisioner-sa

---

kind: ClusterRole # Role of kubernetes
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-clusterRole
rules:
  - apiGroups: [""] # rules on persistentvolumes
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-rolebinding
subjects:
  - kind: ServiceAccount
    name: nfs-pod-provisioner-sa
    namespace: default
roleRef: # binding cluster role to service account
  kind: ClusterRole
  name: nfs-provisioner-clusterRole # name defined in clusterRole
  apiGroup: rbac.authorization.k8s.io

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-pod-provisioner-otherRoles
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]

---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-pod-provisioner-otherRoles
subjects:
  - kind: ServiceAccount
    name: nfs-pod-provisioner-sa # same as top of the file
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: nfs-pod-provisioner-otherRoles
  apiGroup: rbac.authorization.k8s.io
EOT
```

```sh
cat << EOT > pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc-volume
spec:
  storageClassName: nfs-sc # SAME NAME AS THE STORAGECLASS
  accessModes:
    - ReadWriteMany #  must be the same as PersistentVolume
  resources:
    requests:
      storage: 10Gi
EOT

cat << EOT > sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: nfs-sc
provisioner: nfs-provisioner
parameters:
  archiveOnDelete: "false"
EOT
```


## 배포관리 argocd 구성

### 배포관리 Argocd

* namespace 생성

```sh
kubectl create namespace argocd
```

* arogocd 설치

```sh
helm repo add argocd https://argoproj.github.io/argo-helm
helm install argocd argocd/argo-cd --namespace argocd --create-namespace --set server.service.type=LoadBalancer

echo `kubectl get secrets argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d`
```

* argocd cli 설치

```sh
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

* service type 변경 (다른 타입으로 변경 해야 한다면, 그때 사용)

```sh
 k patch svc argocd-server -n argocd -p '{"spec":{"type": "LoadBalancer"}}'
```

* NHN Private 일 경우에만 필요

```sh
 k patch service argocd-server -n argocd -p '{"metadata":{"annotations":{"service.beta.kubernetes.io/openstack-internal-load-balancer":"true"}}}'
```

* 기존 argocd helm 등록

```sh
argocd app create argocd --repo https://argoproj.github.io/argo-helm --helm-chart argo-cd --revision 3.35.4 --dest-server https://kubernetes.default.svc --dest-namespace argocd
```

## LoadBalancing를 위한 metallb 구성

### 클라우드 환경의 K8s LB 와 On-prem 환경 LB의 차이 이해

* ![](Pasted%20image%2020250613092503.png)
* 클라우드 환경의 K8s의 경우 Service Type 으로 LB를 생성 할 경우, 자동으로 IP를 할당 하지만
    On-premises 환경에서는 별도로 로드밸런서를 구성해야 함, MetalLB는 이러한 기능을 담당.
* L2 (ARP), L3(BGP) 방식 모두 지원
* 동작방식
    1. 서비스 생성 및 IP할당
        * 사용자가 LB타입의 서비스를 생성하면, MetalLB가 요청을 감지
        * ConfigMap 에 정의된 IP풀에서 하나의 IP주소를 할당.
    2. IP 주소와 서비스 매핑
        * 할당된 IP주소를 할당 한 후 K8s API서버에 업데이트 함 (해당 IP는 외부에서 접근 가능)
* 외부 네트워크 전파 방식
    1. L2 mode (APR/NDP 기반), L3 mode(BGP)
        1. MetalLB는 선택된 IP주소를 ARP응답으로 브로드 캐스팅 함.
        2. 외부 클러이언트가 이 IP로 요청을 보내면, MetalLB가 리스닝 중인 노드로 전달.
        3. 해당 요청은 K8s 의 서비스 및 Pod 로 라우팅.


### MetalLB

* stricARP 설정 변경 (IP충돌 방지 및 라우팅 문제 해결)

```sh
kubectl edit configmap -n kube-system kube-proxy
```

* metalLB 설치

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

```sh
cat << EOF >> l2.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advert
  namespace: metallb-system
EOF

cat << EOF >> ipaddress.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: production
  namespace: metallb-system
spec:
  addresses:
  - 할당받은 LB IP
EOF
```

* MetalLB 설정에서 고민해봐야 할 점
    * ![](Pasted%20image%2020250616093000.png)
    * ![](Pasted%20image%2020250616093028.png)

## 웹 서비스 구성을 위한 Ingress-nginx

### Ingress-nginx

* dns 신규 신청
* ingress-nginx 구성

```sh
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

![](Pasted%20image%2020250613092638.png)

### ingress 설정

* 기본 ingress 구성

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.toastmaker.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
```

* tls 설정

```yaml
  tls:
  - hosts:
    - argocd-btpa819.toastmaker.net
    secretName: argocd-tls
```

```sh
kubectl create secret tls argocd-tls --cert=/path/to/tls.crt --key=/path/to/tls.key
```

* default tls 설정

```
spec.spec.containers: args 에 
--default-ssl-certificate=default/default-tls
추가
```

## Service Mesh

### istio

* Service Mesh 란?
    * MSA 서비스간 통신을 제어, 관찰, 보안 강화하는데 사용하는 인프라 계층. 마이크로 서비스간의 네트워크 통신을 효율적이고 안전하게 관리 하도록 설계됨.
    * 서비스 간 통신 관리
        * 요청 라우팅, 로드 밸런싱, 서비스 디스커버리 지원.
    * 보안 강화
        * 서비스 간의 통신에 TLS 암호화를 적용하여 안전한 데이터 전송 보장.
        * 서비스 간 인증 및 권한 부여
    * 관찰 및 모니터링
        * 요청 트래픽, 응답 속도, 에러율 등의 실시간 모니터링 가능.
        * 분산 트레이싱 및 로그 수집을 통해 문제 진단.
    * 정책 관리
        * 트래픽 제어 및 액세스 제어 정책 적용.
* Istio 설치

```sh
curl -L https://istio.io/downloadIstio | sh -

ln -s /root/istio-1.24.0/bin/istioctl /usr/local/bin/istioctl

istioctl install --set profile=demo
```

* istio 제거

```
istioctl uninstall --purge
```

### kiali

* kiali 설치

```
k apply -f /root/istio-1.24.0/samples/addons/kiali.yaml
```

# 기타

## 운영을 편리하게 해주는 운영도구

### kubectx, kubens

* kubectx, kubens 설치

```sh
git clone https://github.com/ahmetb/kubectx /opt/kubectx
ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```
### k9s

* 설치

```sh
curl -sS https://webinstall.dev/k9s | bash
cat << EOF >> /etc/profile
## k9s setting
export TERM=xterm-256color 
EOF
```

## cni를 변경해 보자

### calico mode 변경 설정

* vxlan 과 ebpf 설정의 장단점
    ![](Pasted%20image%2020250613092733.png)
* calicoctl install

```sh
curl -L https://github.com/projectcalico/calico/releases/download/v3.29.0/calicoctl-linux-amd64 -o /root/cni/calicoctl
ln -s /root/cni/calicoctl /usr/local/bin/calicoctl
chmod +x /usr/local/bin/calicoctl
```

* k8s 내 IP CIDR 및 IP사용량 확인

```sh
calicoctl ipam show
```

### vxlan > ebpf

* ebpf 변경

```sh
cat << EOF >> ./tigera-ebpf.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: kubernetes-services-endpoint
  namespace: tigera-operator
data:
  KUBERNETES_SERVICE_HOST: 'API server host'
  KUBERNETES_SERVICE_PORT: 'API server port'
EOF
```

* calico-system 모니터링

```sh
watch kubectl get pods -n calico-system
```

* kube-proxy 제거

```sh
kubectl patch ds -n kube-system kube-proxy -p '{"spec":{"template":{"spec":{"nodeSelector":{"non-calico": "true"}}}}}'
kubectl patch felixconfiguration default --patch='{"spec": {"bpfKubeProxyIptablesCleanupEnabled": false}}'
```

* enable bpf mode

```sh
kubectl patch installation.operator.tigera.io default --type merge -p '{"spec":{"calicoNetwork":{"linuxDataplane":"BPF"}}}'
```

* 정리
	* 명확하게 말하면 vxlan 은 터널링 기술 방식이며, ebpf 는 정책처리 방식, 따라서 vxlan과 ebpf 를 동시에 사용 가능 할 수도, 단독으로 ebpf 방식으로만 사용할 수도 있음.
