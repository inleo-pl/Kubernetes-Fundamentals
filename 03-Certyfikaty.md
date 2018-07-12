# Certyfikaty

W tym labie stworzymy infrastrukturę PKI używając CloudFlare's PKI toolkit, cfssl, następnie machniemy Certificate Authority i wygenerujemy certyfikaty TLS dla następujących komponentów: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet i kube-proxy.

Stwórz katalog zawierający certyfikaty na potrzeby klastra Kubernetes:
```
mkdir conf/
cd conf
```
Zdefiniuj zmienne:
```
WORKER01_IP='[worker01-IP]'
WORKER02_IP='[worker02-IP]'
MASTER01_IP='[master01-IP]'
EXTERNAL_IP='[HAProxy-IP]'
```
## Certificate Authority

Wygeneruj plik konfiguracyjny CA, certyfiakty, i prywatny klucz:
```
{

cat > ca-config.json <<EOF
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
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```
## Certyfikaty Client i Server

Wygenerujemy certyfikaty klienta i serwera  każdego komponentu Kubernetesa dla użytkownika admin.

### Certyfikat dla admina

Wygeneruj certyfiakt klancki dla administratora i klucz prywatny:
```
{

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```
### Certyfikaty Kubelet Client

Kolejnym certem jest certyfikat klienta kubelet dla każdego workera, na początku worker01:
```
cat > worker01-csr.json <<EOF
{
  "CN": "system:node:worker01",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=worker01,${WORKER01_IP},${WORKER01_IP} \
  -profile=kubernetes \
  worker01-csr.json | cfssljson -bare worker01
```
Następnie worker02:
```
cat > worker02-csr.json <<EOF
{
  "CN": "system:node:worker02",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=worker02,${WORKER02_IP},${WORKER02_IP} \
  -profile=kubernetes \
  worker02-csr.json | cfssljson -bare worker02
```
### Certyfikat Controller Managera

Wygenerowanie certyfikatu klienckiego dla kube-controller-manager i klucza prywatego:
```
{

cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```
### Certyfiakt Kube Proxy

Kolejnym elementem instalowanmy na węzłach typu worker będzie kube-proxy, ono też potrzebuje certa:
```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```
### Certyfikat Scheduler

Generowanie certyfikatów kube-scheduler i klucza prywatnego:
```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```
### Certyfikat Kubernetes API Server 

Generowanie certyfikatu dla Kubernetes API Server znajdującego się na węzłach typu master:
```
{
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=10.32.0.1,10.240.0.10,10.240.0.11,10.240.0.12,${EXTERNAL_IP},${MASTER01_IP},127.0.0.1,haproxy,master01,kubernetes.localdomain,kubernetes.default \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```
## Service Account Key Pair

Wygeneruj certyfikaty dla service-account i klucz prywatny:
```
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```
## Prześlij plik na serwery

Następnie skopjuj certyfikaty do odpowiednich nodów:
```
scp -i ../Kubernetes_Fundamentals.pem ca.pem worker01-key.pem worker01.pem ubuntu@${WORKER01_IP}:~
scp -i ../Kubernetes_Fundamentals.pem ca.pem worker02-key.pem worker02.pem ubuntu@${WORKER02_IP}:~
scp -i ../Kubernetes_Fundamentals.pem ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem ubuntu@${MASTER01_IP}:~
```
