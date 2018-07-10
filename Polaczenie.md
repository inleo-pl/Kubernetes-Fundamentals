Pobierz klucz
```
wget https://raw.githubusercontent.com/inleo-pl/Warsztaty-Kubernetes-Fundamentlas/master/Kubernetes_Fundamentals.pem
```
Nadaj uprawnienia i połącz się:
```
chmod 600 /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@haproxy
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@manager01
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@worker01
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@worker02
```
