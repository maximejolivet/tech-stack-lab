# Exercices Kubernetes — Niveau 3 (Avancé)

## Exercice 1 — StatefulSet vs Deployment

Explique en 2-3 phrases pourquoi une base de données Postgres en cluster serait déployée via un `StatefulSet` plutôt qu'un `Deployment` classique.

## Exercice 2 — RBAC

Écris un `Role` (namespace `staging`) qui autorise uniquement les verbes `get` et `list` sur les Pods, et un `RoleBinding` qui l'associe à un utilisateur `alice`.

## Exercice 3 — Network Policy

Écris une `NetworkPolicy` qui n'autorise que les Pods labellisés `app: mon-api` à se connecter au Pod `db` (labellisé `app: db`) sur le port 5432, et bloque tout le reste du trafic entrant vers `db`.

## Exercice 4 — Helm

Explique en 2-3 phrases ce qu'apporte un chart Helm par rapport à un ensemble de manifestes YAML bruts, et donne la commande pour installer PostgreSQL depuis le repo Bitnami.

## Exercice 5 — GitOps

Décris en 3-4 phrases le principe de GitOps (avec un outil comme ArgoCD) appliqué à ce cluster : où vit la source de vérité, comment un changement de manifeste se propage jusqu'au cluster, et quel avantage cela apporte par rapport à des `kubectl apply` manuels.
