# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: mon-api-config }
data:
  LOG_LEVEL: "info"
---
apiVersion: v1
kind: Secret
metadata: { name: mon-api-secret }
type: Opaque
stringData:
  DB_PASSWORD: "secret"
```

```yaml
# extrait du Deployment
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef: { name: mon-api-config, key: LOG_LEVEL }
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef: { name: mon-api-secret, key: DB_PASSWORD }
```

## Exercice 2

```yaml
livenessProbe:
  httpGet: { path: /health, port: 3000 }
  periodSeconds: 10
readinessProbe:
  httpGet: { path: /ready, port: 3000 }
  periodSeconds: 5
```

## Exercice 3

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mon-api-ingress
spec:
  rules:
    - host: mon-api.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: mon-api-svc
                port:
                  number: 80
```

## Exercice 4

```bash
kubectl set image deployment/mon-api api=mon-api:2.0
kubectl rollout status deployment/mon-api
kubectl rollout undo deployment/mon-api
```

## Exercice 5

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mon-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mon-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```
