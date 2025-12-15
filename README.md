# A. Utworzenie klastra
```
minikube start --nodes=4 -p lab --cni=calico --container-runtime=docker
```
# B. Utworzenie obiektów środowiska
## Tworzenie przestrzeni nazw
```
kubectl create -f namespaces.yaml
```
## 1. Deployment frontend
```
kubectl create -f deployment-frontend.yaml
```
## 2. Deployment backend
```
kubectl create -f deployment-backend.yaml
```
## 3. Pod my-sql
```
kubectl create -f pod-mysql.yaml
```
## 4. Service frontend
```
kubectl create -f service-frontend.yaml
```
## 5. Services backend oraz my-sql
```
kubectl create -f service-backend.yaml
kubectl create -f service-mysql.yaml
```
# C. Polityka sieciowa
## 1. Dostęp z frontend do backend tylko na port 80
```
kubectl create -f networkpolicy-frontend.yaml
```
## 2. Dostęp z backend do my-sql tylko na port 3306 oraz dostęp do frontend 
```
kubectl create -f networkpolicy-backend.yaml
```
# D. Ograniczenia Resource Quota
```
kubectl create -f resourcequotas.yaml
```
# E. HPA dla frontend
```
kubectl create -f hpa.yaml
```
# G. Test poprawności konfiguracji przy obciążeniu
## Uruchomiamy pod generujący obciążanie
```
 kubectl run -it load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://frontend.frontend.svc.cluster.local; done" -n default
```
<img width="1304" height="680" alt="image" src="https://github.com/user-attachments/assets/566aaf99-d26b-4267-b38a-06e1042306d2" />

## Sprawdzamy skalowanie
```
kubectl get hpa -n frontend -w
```
<img width="1304" height="680" alt="image" src="https://github.com/user-attachments/assets/2845fc2e-b371-4041-8f67-55cff3dacadf" />

#### po usunięciu load generatora
<img width="1304" height="680" alt="image" src="https://github.com/user-attachments/assets/a6aecd5f-73b5-4720-8ed3-2f26702309d4" />



