# Pliki konfiguracyjne

## Pliki konfiguracyjne kubelet

Musimy skonfigurować kubelet na każdym węźle klastra typu worker, na początku generujemy kubeconfig dla kubelets. Na poczatku worker01:
```
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=worker01.kubeconfig

  kubectl config set-credentials system:node:worker01 \
    --client-certificate=worker01.pem \
    --client-key=worker01-key.pem \
    --embed-certs=true \
    --kubeconfig=worker01.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker01 \
    --kubeconfig=worker01.kubeconfig

  kubectl config use-context default --kubeconfig=worker01.kubeconfig
}
```
Następnie worker02:
```
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=worker02.kubeconfig

  kubectl config set-credentials system:node:worker02 \
    --client-certificate=worker02.pem \
    --client-key=worker02-key.pem \
    --embed-certs=true \
    --kubeconfig=worker02.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker02 \
    --kubeconfig=worker02.kubeconfig

  kubectl config use-context default --kubeconfig=worker02.kubeconfig
}
```
## Pliki konfiguracyjne kube-proxy

Tworzymy następnie konfig dla kube-proxy:
```
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```
## Pliki konfiguracyjne kube-controller-manager

Tworzymy pliki konfiguracyjne dla kube-controller-manager:
```
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```
## Pliki konfiguracyjne kube-scheduler

Tworzymy pliki konfiguracyjne dla kube-scheduler:
```
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```
## Pliki konfiguracyjne admin

Tworzymy pliki konfiguracyjne dla kube-scheduler:
```
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```
## Prześlij plik na serwery

Następnie skopjuj certyfikaty do odpowiednich nodów:
```
scp -i ../Kubernetes_Fundamentals.pem worker01.kubeconfig kube-proxy.kubeconfig ubuntu@${WORKER01_IP}:~
scp -i ../Kubernetes_Fundamentals.pem worker02.kubeconfig kube-proxy.kubeconfig ubuntu@${WORKER02_IP}:~
scp -i ../Kubernetes_Fundamentals.pem admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ubuntu@${MASTER01_IP}:~
```
