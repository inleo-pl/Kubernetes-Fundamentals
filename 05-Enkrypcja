Kubernetes przechowuje różne dane - stan klastra, konfigurację aplikacji i sekrety. Zaszyfrujmy to!

## Encryption Key
Tworzymy klucz szyfrowania dla secretów:
```
sudo ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
## Encryption Config File

Manifest dla konfiguracji szyfrowania K8S:
```
cat > encryption-config.yaml <<EOF
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
scp -i ../Kubernetes_Fundamentals.pem encryption-config.yaml ubuntu@${MASTER01_IP}:~
```
