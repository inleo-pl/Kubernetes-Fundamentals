## Uruchomienie etcd

Wszytskie elementy Kubernetesa są stateless. Jedyne miejsce gdzie trzymane są dane to etcd. Należy go uruchomić na każdym węźle typu master. W labie mamy jeden, jednak jak byście chcieli dołaczyć więcej w celu uzyskania niezawodności należy odpalić poniższe polecenia na każdym kolejnym węźle.

Przygotowaliśmy już sporo elementów, ale nasze środowisko wciąż wygląda ubogo... ale niemartw się za chwilę dorzucimy do niego trochę elementów, tymczasem jesteśmy tu:

![K8S-Architecture-Diagram-HA-Proxy](https://inleo.pl/wp-content/uploads/2018/08/K8S-Architecture-Diagram-HAProxy.png)

Logujemy się na węzeł master01 i sciągamy niezbędne pliki:
```
wget -q --show-progress --https-only --timestamping "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
```
Rozpakowujemy je:
```
tar xvzf etcd-v3.3.5-linux-amd64.tar.gz
```
Przenosimy do docelowej lokalizacji:
```
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
```
Tworzymy folder konfguracji:
```
sudo mkdir -p /etc/etcd /var/lib/etcd
```
Kopjujemy certy:
```
cd /home/ubuntu/
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```
Definujemy zmienne:
```
ETCD_NAME=$(hostname -s)
INTERNAL_IP=$(hostname -i)
```
Tworzymy plik konfiguracji:
```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${ETCD_NAME}=https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
Przeładuj serwis etcd i uruchom na starcie:
```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```
Zweryfikuj czy etcd działa:
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```
Yey! Mamy etcd na pokładzie, nasze środowisko wygląda teraz tak:

![K8S-Architecture-Diagram-etcd](https://inleo.pl/wp-content/uploads/2018/08/K8S-Architecture-Diagram-etcd.png)
