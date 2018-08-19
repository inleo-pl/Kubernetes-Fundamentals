# Połączenie

Pobierz klucz na swój komputer:
```
wget https://raw.githubusercontent.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/master/Kubernetes_Fundamentals.pem
```
Nadaj uprawnienia i połącz się:
```
chmod 600 /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@haproxy
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@manager01
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@worker01
ssh -i /miejsce/gdzie/jest/Kubernetes_Fundamentals.pem ubuntu@worker02
```
Nasz lab docelowo będzie wyglądał tak:

![K8S-Architecture-Diagram-Overview](https://inleo.pl/wp-content/uploads/2018/08/K8S-Architecture-Diagram-Overview.png)

Tymczasem, zamin do tego dojdzie sprawdź IP każdego z powyższych serwerów i zapisz je sobie. Dodatkowo sciągnij sobie na HAProxy certyfikat .pem:
```
wget https://raw.githubusercontent.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/master/Kubernetes_Fundamentals.pem
chmod 600 Kubernetes_Fundamentals.pem
```
## Podsumowanie
Wiemy gdzie biegniemy, jesteśmy połączeni! Zatem czas na następny moduł [HAProxy](https://github.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/blob/master/02-HAProxy.md).
