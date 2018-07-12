# Serwisy
Wyświetlamy serwisy:
```
kubectl get services
```
Powołaj serwis:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
Dodawanie użytkowika, który będzie miał dostęp do Dashbord:
```
vi kubernetes-dashboard-admin.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```
Dodajemy użytkownika:
```
kubectl create -f kubernetes-dashboard-admin.yaml
```
Pozykujemy token:
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
Edytujmy konfig, aby dopiąć się do NodePort:
```
kubectl -n kube-system edit service kubernetes-dashboard

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-07-11T23:08:28Z
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "4075"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard
  uid: 525509f7-855f-11e8-863e-2cc260628564
spec:
  clusterIP: 10.32.0.140
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
Wymieniamy type: ClusterIP na type: NodePort. Następnie
```
kubectl -n kube-system get service kubernetes-dashboard
```
Zaktualziuj HAProxy i wejdz na stronkę... nie zapomnij o tym że masz iśc przez https://zewnetrzny.adrestwojego.ha.proxy:8080
