Tworzymy klucz szyfrowania dla secretów:
```
sudo ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
Manifest dla konfiguracji szyfrowania K8S:
```
sudo cat > encryption-config.yaml << EOF

kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
Kopjujemy manifest do serwera master01:
```
scp encryption-config.yaml [master01-IP]:~
```
