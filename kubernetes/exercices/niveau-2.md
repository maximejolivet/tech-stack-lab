# Exercices Kubernetes — Niveau 2 (Intermédiaire)

## Exercice 1 — ConfigMap et Secret

Écris une `ConfigMap` `mon-api-config` avec la clé `LOG_LEVEL: "info"`, et un `Secret` `mon-api-secret` avec la clé `DB_PASSWORD`. Modifie le Deployment de l'exercice 2 (niveau 1) pour injecter ces deux valeurs en variables d'environnement dans le conteneur.

## Exercice 2 — Probes

Ajoute au Deployment `mon-api` une `livenessProbe` et une `readinessProbe` HTTP sur les chemins respectifs `/health` et `/ready`, port 3000.

## Exercice 3 — Ingress

Écris un `Ingress` qui route le domaine `mon-api.local` vers le Service `mon-api-svc` sur le port 80.

## Exercice 4 — Rolling update et rollback

Explique la commande pour déclencher une mise à jour de l'image du Deployment `mon-api` vers `mon-api:2.0`, celle pour suivre l'état du déploiement, et celle pour revenir à la version précédente si un problème est détecté.

## Exercice 5 — Horizontal Pod Autoscaler

Écris un `HorizontalPodAutoscaler` qui fait varier le nombre de répliques du Deployment `mon-api` entre 2 et 10, ciblant 70% d'utilisation CPU moyenne.
