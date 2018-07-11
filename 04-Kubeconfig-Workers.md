Stwórz katalog:
```
cd ..
mkdir config
cd config/
```
Musimy skonfigurować kubelet i kube-proxy na każdym węźle klastra typu worker, na początku generujemy kubeconfig dla kubelets. Zaczynamy od informacji dot klastra:
```
kubectl config set-cluster kubernetes \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://[HAProxy-IP]:6443 \
--kubeconfig='[worker01-IP].kubeconfig'
```
Następnie dodoajemy uprawnienia:
```
kubectl config set-credentials system:node:[worker01-IP] \
--client-certificate=[worker01-IP].pem \
--client-key=[worker01-IP]-key.pem \
--embed-certs=true \
--kubeconfig='[worker01-IP].kubeconfig'
```
I dorzucemy kontekst:
```
kubectl config set-context default \
--cluster=kubernetes \
--user=system:node:[worker01-IP] \
--kubeconfig='[worker01-IP].kubeconfig'

kubectl config use-context default --kubeconfig=[worker01-IP].kubeconfig
```
Ponawiamy wszytskie kroki dla węzła drugiego:
```
kubectl config set-cluster kubernetes \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://[HAProxy-IP]:6443 \
--kubeconfig='[worker02-IP].kubeconfig'
```
Następnie dodoajemy uprawnienia:
```
kubectl config set-credentials system:node:[worker02-IP] \
--client-certificate=[worker02-IP].pem \
--client-key=[worker02-IP]-key.pem \
--embed-certs=true \
--kubeconfig='[worker02-IP].kubeconfig'
```
I dorzucemy kontekst:
```
kubectl config set-context default \
--cluster=kubernetes \
--user=system:node:[worker02-IP] \
--kubeconfig='[worker02-IP].kubeconfig'

kubectl config use-context default --kubeconfig=[worker02-IP].kubeconfig
```
Tworzymy następnie konfig dla kube-proxies:
```
kubectl config set-cluster kubernetes \
--certificate-authority=ca.pem \
--embed-certs=true \
--server=https://[HAProxy-IP]:6443 \
--kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials kube-proxy \
--client-certificate=kube-proxy.pem \
--client-key=kube-proxy-key.pem \
--embed-certs=true \
--kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
--cluster=kubernetes \
--user=kube-proxy \
--kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```
Kopjujemy wszytsko na odpowiednie hosty:
```
scp [worker01-IP].kubeconfig kube-proxy.kubeconfig [worker01-IP]:~
scp [worker02-IP].kubeconfig kube-proxy.kubeconfig [worker02-IP]:~
```
