Połącz się z worker01 i wyłącz swap:
```
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
Zmodyfikuj hostname aby pasował do IP:
```
sudo hostnamectl set-hostname 10.10.40.70
```
Zaktualizuj system:
```
sudo apt-get update
sudo apt-get -y upgrade
```
Zainstaluj socat (dla port-forwarding):
```
sudo apt-get install socat
```
Ściągnij binarki:
```
wget https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz
wget https://github.com/kubernetes-incubator/cri-containerd/releases/download/v1.0.0-beta.0/cri-containerd-1.0.0-beta.0.linux-amd64.tar.gz
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kube-proxy
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubelet
```
Stwórz foldery instalacyjne:
```
sudo mkdir -p \
/etc/cni/net.d \
/opt/cni/bin \
/var/lib/kubelet \
/var/lib/kube-proxy \
/var/lib/kubernetes \
/var/run/kubernetes
```
Zainstaluj:
```
sudo tar -xvzf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/
sudo tar -xvzf cri-containerd-1.0.0-beta.0.linux-amd64.tar.gz -C /
chmod +x kubectl kube-proxy kubelet
sudo mv kubectl kube-proxy kubelet /usr/local/bin/
```
Przekonfiguruj CNI:
```
sudo vi /etc/cni/net.d/10-bridge.conf

{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "10.20.0.0/24"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}

sudo vi /etc/cni/net.d/99-loopback.conf

{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
```
Skonfiguruj kubelet:
```
sudo cp [worker-01]-key.pem [worker-01].pem /var/lib/kubelet
sudo cp [worker-01].kubeconfig /var/lib/kubelet/kubeconfig
sudo cp ca.pem /var/lib/kubernetes
```
Stwórz plik systemd dla kubeleta:
```
sudo vi /etc/systemd/system/kubelet.service

[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=cri-containerd.service
Requires=cri-containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \
  --allow-privileged=true \
  --anonymous-auth=false \
  --authorization-mode=Webhook \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --cloud-provider= \
  --cluster-dns=10.32.0.10 \
  --cluster-domain=cluster.local \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/cri-containerd.sock \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --pod-cidr=10.20.0.0/24 \
  --register-node=true \
  --runtime-request-timeout=15m \
  --tls-cert-file=/var/lib/kubelet/[worker01-IP].pem \
  --tls-private-key-file=/var/lib/kubelet/[worker01-IP]-key.pem \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Skonfiguruj kube-proxy:
```
sudo cp kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```
Stwórz systemd dla kube-proxy:
```
sudo vi /etc/systemd/system/kube-proxy.service

[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --cluster-cidr=10.20.0.0/16 \
  --kubeconfig=/var/lib/kube-proxy/kubeconfig \
  --proxy-mode=iptables \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Przeładuj, uruchom przy starcie i uruchom serwisy:
```
sudo systemctl daemon-reload
sudo systemctl enable containerd cri-containerd kubelet kube-proxy
sudo systemctl start containerd cri-containerd kubelet kube-proxy
```
Powtórz dla drugiego węzła (worker02).
