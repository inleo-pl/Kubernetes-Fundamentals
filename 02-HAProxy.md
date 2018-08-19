# HAproxy

Zaczynamy od HA Proxy. Nasze środowisko jest puste i wygłada następująco:

![K8S-Architecture-Diagram-Empty](https://inleo.pl/wp-content/uploads/2018/08/K8S-Architecture-Diagram-Empty.png)

Należy zaktualizować serwer HAProxy:
```
sudo apt-get update
sudo apt-get -y upgrade
```
Kolejnym krokiem jest instalacja narzedzi, które normalnie instaluje się na końcówce z której konfgurujemy agenta, ale teraz użyjemy serwera HAProxy.
Następnie instalujemy Cloud Flare SSL:
```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssl*
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```
Weryfikujemy instlację:
```
cfssl version
```
Instalujemy kubectl:
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin
kubectl version
```
Czas zainstalować HAProxy:
```
sudo apt-get -y install haproxy
sudo vi /etc/haproxy/haproxy.cfg
```
W konfigurację HAProxy dopisujemy na końcu następujęce linie następujące parametry:
```
global

defaults

frontend kubernetes
    bind        [HAProxy-IP]:6443
    option      tcplog
    mode        tcp
    default_backend kubernetes-master-nodes


backend kubernetes-master-nodes
    mode    tcp
    balance roundrobin
    option  tcp-check
    server  master01 [master01-IP]:6443 check fall 3 rise 2
```
Każda kolejna linia server oznaczałaby kolejny serwer master K8S, w naszym przypadku mamy tylko jeden. Starujemy HAProxy
```
sudo systemctl restart haproxy
```
