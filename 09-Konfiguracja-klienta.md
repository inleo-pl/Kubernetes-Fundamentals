# Konfiguracja kliena
Połącz się z HAProxy i skonfigueruj jako maszynę kliencką, aby miała dostęp do klastra:
```
EXTERNAL_IP=$(hostname -i)
cd conf/
```
Przekonfiguruj klienta
```
kubectl config set-cluster kubernetes \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${EXTERNAL_IP}:6443
```
Dodaj uprawnienia:
```
kubectl config set-credentials admin \
--client-certificate=admin.pem \
--client-key=admin-key.pem
```
Kontekst:
```
kubectl config set-context kubernetes --cluster=kubernetes --user=admin
```
Skonfiguruj klienta:
```
kubectl config use-context kubernetes
```
Sprawdz:
```
kubectl get nodes
```
