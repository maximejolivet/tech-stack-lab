# Solutions — Niveau 3 (Avancé)

## Exercice 1

Un `StatefulSet` donne à chaque réplique une identité stable (nom prévisible, ex. `db-0`, `db-1`) et un volume persistant qui lui reste attaché même après un redémarrage — indispensable pour un cluster Postgres où chaque nœud a un rôle distinct (primaire/réplicas) et des données propres. Un `Deployment` traite ses Pods comme strictement interchangeables et sans identité durable, ce qui convient à une application sans état comme une API, mais casserait la cohérence d'un cluster de base de données.

## Exercice 2

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: staging
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-pod-reader
  namespace: staging
subjects:
  - kind: User
    name: alice
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Exercice 3

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
spec:
  podSelector:
    matchLabels: { app: db }
  policyTypes: ["Ingress"]
  ingress:
    - from:
        - podSelector:
            matchLabels: { app: mon-api }
      ports:
        - port: 5432
```

## Exercice 4

Un chart Helm empaquette un ensemble de manifestes YAML sous forme de templates paramétrables (via un fichier `values.yaml`), versionnés et réutilisables — au lieu d'écrire et maintenir à la main tous les manifestes d'une application complexe (Deployment, Service, ConfigMap, Secret, PVC...), on installe/configure/met à jour le tout en une commande, avec gestion native des releases et des rollbacks.

```bash
helm install ma-db bitnami/postgresql
```

## Exercice 5

Dans une approche GitOps, un dépôt Git contient la description déclarative de l'état désiré du cluster (les manifestes YAML) et fait autorité : aucun `kubectl apply` manuel n'est censé modifier directement le cluster. Un outil comme ArgoCD surveille en continu ce dépôt, détecte tout écart entre l'état déclaré en Git et l'état réel du cluster, et applique automatiquement la synchronisation nécessaire. Un changement (ex. nouvelle image, nouvelle réplique) se propage donc simplement via une pull request mergée sur le dépôt de manifestes, sans accès direct au cluster. L'avantage principal est la traçabilité (chaque changement du cluster est un commit Git, review-able et revertable) et la cohérence garantie entre ce qui est documenté et ce qui tourne réellement.
