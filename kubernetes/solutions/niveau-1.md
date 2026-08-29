# Solutions — Niveau 1 (Bases)

## Exercice 1

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mon-api
spec:
  containers:
    - name: api
      image: mon-api:1.0
      ports:
        - containerPort: 3000
```

## Exercice 2

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-api
spec:
  replicas: 3
  selector:
    matchLabels: { app: mon-api }
  template:
    metadata:
      labels: { app: mon-api }
    spec:
      containers:
        - name: api
          image: mon-api:1.0
          ports: [{ containerPort: 3000 }]
```

## Exercice 3

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mon-api-svc
spec:
  selector: { app: mon-api }
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

## Exercice 4

```bash
kubectl get pods
kubectl describe pod mon-api-abc123
kubectl logs mon-api-abc123
```

## Exercice 5

```bash
kubectl create namespace staging
kubectl apply -f deployment.yaml -n staging
```
