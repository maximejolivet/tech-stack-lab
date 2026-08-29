# Exercices Kubernetes — Niveau 1 (Bases)

## Exercice 1 — Pod minimal

Écris un manifeste `pod.yaml` pour un Pod nommé `mon-api` exécutant l'image `mon-api:1.0`, exposant le port 3000.

## Exercice 2 — Deployment

Écris un `Deployment` nommé `mon-api` avec 3 répliques, à partir de l'image `mon-api:1.0`, port 3000.

## Exercice 3 — Service

Écris un `Service` de type `ClusterIP` nommé `mon-api-svc` qui expose le Deployment de l'exercice 2 sur le port 80 (routé vers le port 3000 des Pods).

## Exercice 4 — Commandes kubectl de base

Donne les commandes pour : lister tous les Pods, afficher les détails et événements d'un Pod nommé `mon-api-abc123`, et afficher ses logs.

## Exercice 5 — Namespace

Crée un namespace `staging`, puis applique le Deployment de l'exercice 2 dans ce namespace.
