# Konfiguracja klienta
Wykożystamy HA Proxy jako serwer do zarządzania naszym klastrem. Dla przypomnienia infra wygląda teraz tak:

![K8S-Architecture-Diagram-Worker-Nodes](https://inleo.pl/wp-content/uploads/2018/08/K8S-Architecture-Diagram-Worker-Nodes.png)

Połącz się zatem z HAProxy i skonfiguruj jako maszynę kliencką, aby miała dostęp do klastra:
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
Dodajemy uprawnienia dla naszego certu wystawionego na użytkownika kubernetes:
```
kubectl create clusterrolebinding apiserver-kubelet-api-admin --clusterrole system:kubelet-api-admin --user kubernetes
```
## Podsumowanie
Masz swój pierwszy działający klaster, który postawiłeś od początku do końca sam. Gratulacje! Tymczasem Twoja infra jest już całkiem imponująca:

![K8S-Architecture-Diagram-Client](https://inleo.pl/wp-content/uploads/2018/08/K8S-Architecture-Diagram-Client.png)

Czas puścić na nim jakieś aplikacje!
