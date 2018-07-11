Logujemy się na węzeł master01 i sciągamy niezbędne pliki:
```
wget https://github.com/coreos/etcd/releases/download/v3.2.11/etcd-v3.2.11-linux-amd64.tar.gz
```
Rozpakowujemy je:
```
tar xvzf etcd-v3.2.11-linux-amd64.tar.gz
```
Przenosimy do docelowej lokalizacji:
```
sudo mv etcd-v3.2.11-linux-amd64/etcd* /usr/local/bin/
```
Tworzymy folder konfguracji:
```
sudo mkdir -p /etc/etcd /var/lib/etcd
```
Kopjujemy certy:
```
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```
Tworzymy plik konfiguracji:
```
sudo vi /etc/systemd/system/etcd.service

[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \
  --name [master01-IP] \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://[master01-IP]:2380 \
  --listen-peer-urls https://[master01-IP]:2380 \
  --listen-client-urls https://[master01-IP]:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://[master01-IP]:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster [master01-IP]=https://[master01-IP]:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Przeładuj serwis etcd i uruchom na starcie:
```
sudo systemctl enable etcd
sudo systemctl start etcd
```
Zweryfikuj czy etcd działa:
```
ETCDCTL_API=3 etcdctl member list
```
