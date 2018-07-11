Połącz się z serwerem master01 i ściągnij niezbędne rzeczy:
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kube-apiserver
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kube-controller-manager
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kube-scheduler
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl
```
Nadaj uprawnienia:
```
sudo chmod +x \
kube-apiserver \
kube-controller-manager \
kube-scheduler \
kubectl
```
Przenieś w miejsce docelowe:
```
sudo mv \
kube-apiserver \
kube-controller-manager \
kube-scheduler \
kubectl \
/usr/local/bin/
```
Stwórz katalog z konfiguracją:
```
sudo mkdir -p /var/lib/kubernetes
```
Skopjuj certy i konfigurację API:
```
sudo cp \
ca.pem \
ca-key.pem \
kubernetes-key.pem \
kubernetes.pem \
encryption-config.yaml \
/var/lib/kubernetes
```
Stwórz konfig API systemd:
```
sudo vi /etc/systemd/system/kube-apiserver.service

[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --admission-control=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
  --advertise-address=[maser01-IP] \
  --allow-privileged=true \
  --apiserver-count=3 \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --enable-swagger-ui=true \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \
  --etcd-servers=https://[maser01-IP]:2379 \
  --event-ttl=1h \
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
  --insecure-bind-address=127.0.0.1 \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \
  --kubelet-https=true \
  --runtime-config=api/all \
  --service-account-key-file=/var/lib/kubernetes/ca-key.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-ca-file=/var/lib/kubernetes/ca.pem \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Stwórz plik controler manager systemd:
```
sudo vi /etc/systemd/system/kube-controller-manager.service

[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --address=0.0.0.0 \
  --cluster-cidr=10.20.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \
  --leader-elect=true \
  --master=http://127.0.0.1:8080 \
  --root-ca-file=/var/lib/kubernetes/ca.pem \
  --service-account-private-key-file=/var/lib/kubernetes/ca-key.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Tworzymy konfiguracje Schdulera:
```
sudo vi /etc/systemd/system/kube-scheduler.service

[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --leader-elect=true \
  --master=http://127.0.0.1:8080 \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Przeładuj konfigurację demonów, uruchom przy starcie systemu i odpal:
```
sudo systemctl daemon-reload

sudo systemctl enable \
kube-apiserver \
kube-controller-manager \
kube-scheduler

sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```
Sprawdz czy działa:
```
kubectl get componentstatuses
```
Skonfigurujmy API dla master01, tak aby mogło uzyskać dostęp do kubeleta, na początku manifest:
```
sudo vi kube-apiserver-to-kubelet.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"

sudo kubectl create -f kube-apiserver-to-kubelet.yaml
```
Następnie manifest który pozoli przypiąć nam rolę klastra do użytkownika poprzez API:
```
sudo vi kube-apiserver-to-kubelet-bind.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
    
sudo kubectl create -f kube-apiserver-to-kubelet-bind.yaml
```
Sprawdzamy:
```
curl --cacert ca.pem https://[HAProxy-IP:6443/version
```