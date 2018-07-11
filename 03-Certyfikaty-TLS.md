Stwórz katalog zawierający certyfikaty na potrzeby klastra Kubernetes:
```
mkdir cert/
cd cert
```
Plik konfiguracyjny CA:
```
sudo vi ca-config.json

{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
```
Następnie CA signing request:
```
sudo vi ca-csr.json

{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IE",
      "L": "Cork",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Cork Co."
    }
  ]
}

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
Czas na certy dla K8S, na start certyfikat dla klienta (admina):
```
sudo vi admin-csr.json

{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IE",
      "L": "Cork",
      "O": "system:masters",
      "OU": "Kubernetes",
      "ST": "Cork Co."
    }
  ]
}

cfssl gencert \
-ca=ca.pem \
-ca-key=ca-key.pem \
-config=ca-config.json \
-profile=kubernetes admin-csr.json | \
cfssljson -bare admin
```
Kolejnym certem jest certyfikat klienta kubelet dla każdego workera, na początku worker01:
```
sudo vi [IP-worker-1]-csr.json

{
  "CN": "system:node:[IP-worker-1]",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IE",
      "L": "Cork",
      "O": "system:nodes",
      "OU": "Kubernetes",
      "ST": "Cork Co."
    }
  ]
}

cfssl gencert \
-ca=ca.pem \
-ca-key=ca-key.pem \
-config=ca-config.json \
-hostname=[IP-worker-1],[IP-worker-1] \
-profile=kubernetes [IP-worker-1]-csr.json | \
cfssljson -bare [IP-worker-1]
```
Następnie worker02:
```
sudo vi [IP-worker-2]-csr.json

{
  "CN": "system:node:[IP-worker-2]",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IE",
      "L": "Cork",
      "O": "system:nodes",
      "OU": "Kubernetes",
      "ST": "Cork Co."
    }
  ]
}

cfssl gencert \
-ca=ca.pem \
-ca-key=ca-key.pem \
-config=ca-config.json \
-hostname=[IP-worker-2],[IP-worker-2] \
-profile=kubernetes [IP-worker-2]-csr.json | \
cfssljson -bare [IP-worker-2]
```
Kolejnym elementem instalowanmy na węzłach typu worker będzie kube-proxy, ono też potrzebuje certa:
```
sudo vi kube-proxy-csr.json

{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IE",
      "L": "Cork",
      "O": "system:node-proxier",
      "OU": "Kubernetes",
      "ST": "Cork Co."
    }
  ]
}

cfssl gencert \
-ca=ca.pem -ca-key=ca-key.pem \
-config=ca-config.json \
-profile=kubernetes kube-proxy-csr.json | \
cfssljson -bare kube-proxy
```
Ostatniem modułem potrzebującem certa jest API znajdujące się na węzłach typu master:
```
sudo vi kubernetes-csr.json

{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IE",
      "L": "Cork",
      "O": "Kubernetes",
      "OU": "Kubernetes",
      "ST": "Cork Co."
    }
  ]
}

cfssl gencert \
-ca=ca.pem \
-ca-key=ca-key.pem \
-config=ca-config.json \
-hostname=[master01-IP],kubernetes.default,master01 \
-profile=kubernetes kubernetes-csr.json | \
cfssljson -bare kubernetes
```
Następnie skopjuj certyfikaty do odpowiednich nodów:
```
scp -i ../Kubernetes_Fundamentals.pem ca.pem [worker01-IP]-key.pem [worker01-IP].pem ubuntu@[worker01-IP]:~
scp -i ../Kubernetes_Fundamentals.pem ca.pem [worker02-IP]-key.pem [worker02-IP].pem [worker02-IP]:~
scp -i ../Kubernetes_Fundamentals.pem ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem [master01-IP]:~
```
