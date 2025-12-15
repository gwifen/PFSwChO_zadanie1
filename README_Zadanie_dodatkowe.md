# 1. Odpowiedź: 
Tak, kubernetes pozwala dokonanie aktualizacji aplikacji, gdy aplikacja jest pod kontrolą autoskalera.
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#autoscaling-during-rolling-update
# 2. Przykladowa strategia Rolling Update
```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```
## a) Zawsze aktywne co najmniej 2 pody

Deployment ma `replicas: 3`  
`maxUnavailable: 1` oznacza, że maksymalnie jeden Pod może być niedostępny podczas aktualizacji  
W najgorszym przypadku:  
3 - 1 = 2 aktywne Pody
## b,c) Przekroczonie wcześniej zdeﬁniowanych parametrów ograniczeń
#### Obecna Resource Quota:
```
pods: "10"
requests.cpu: "1000m"
requests.memory: 1500Mi
```

#### Zużycie przy 10 Podach:
CPU: 10 × 100m = 1000m (limit)  
RAM: 10 × 128Mi = 1280Mi (ok)  
Pody: 10 (ok)  
  
Ale istnieje możliwość chwilowego przekroczenia limitu podów:  
10 (HPA) + 1 (maxSurge) = 11 Podów  
Oraz ustawiony nizki próg `averageUtilization: 10` dla tego, żeby pokazać, że następuje automatyczne skalowanie, więc zmienimy te parametry  
#### Zmodyfikowany manifest HPA
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
  namespace: frontend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 3
  maxReplicas: 9
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
