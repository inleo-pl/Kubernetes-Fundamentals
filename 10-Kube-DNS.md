# Kube DNS
Uruchamiamy serwis DNS.
```
kubectl create -f https://raw.githubusercontent.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/master/kube-dns.yaml
```
Zobaczmy co nam się urodziło:
```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```
Zwerifikujmy czy działa nam DNS
```
kubectl run busybox --image=busybox --command -- sleep 3600
kubectl get pods -l run=busybox
```
Pobierz pełną nazwę busyboxa:
```
kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}"
```
Wprowadz uzyskaną nazwę ponizej:
```
kubectl exec -ti [nazwa] -- nslookup kubernetes
```
## Podsumowanie
Odpaliliśmy coś, ale o co chodzi? Serwisy, deploymenty, pody... nie rozumiem! Zacznijmy zatem od [Podów](https://github.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/blob/master/11-Pody.md).
