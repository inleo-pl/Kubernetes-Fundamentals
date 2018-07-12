# Deployment

Stwórz plik:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
Następnie wywołaj deployment:
```
kubectl create -f  https://k8s.io/examples/controllers/nginx-deployment.yaml
```
Wypisz aktualne deploymenty:
```
kubectl get deployments
```
Sprawdz aktualny status deploymentu:
```
kubectl rollout status deployment/nginx-deployment
```
Przejżyj lebels:
```
kubectl get pods --show-labels
```
Update:
```
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
```
Opis:
```
kubectl describe deployments
```
Rollback, można rozpocząc od psrawdzenia historii deploymentów:
```
kubectl rollout history deployment/nginx-deployment
```
Wybranie np. rev 2 i powrót do niego:
```
kubectl rollout history deployment/nginx-deployment --revision=2
```
Lub do ostatniej:
```
kubectl rollout undo deployment/nginx-deployment
```
Skalowanie:
```
kubectl scale deployment nginx-deployment --replicas=10
```
Autoskalowanie:
```
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
```
