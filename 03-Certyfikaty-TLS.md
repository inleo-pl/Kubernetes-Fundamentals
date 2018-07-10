Plik konfiguracyjny CA:
```
sudo vi ca-config.json
```
W plik wstawiamy:
```
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
```
Wprowadzamy:
```
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
```
Generujemy certy:
```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
ls -la
```
Czas na certy dla K8S, na start dla klienta (admina):
```
sudo vi admin-csr.json
```
Wstawiamy:
```
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
```
