# Exercices AWS — Niveau 2 (Intermédiaire)

## Exercice 1 — Fonction Lambda minimale

Écris une fonction Lambda (Node.js ou Python, au choix) qui retourne `"Bonjour, <nom> !"` à partir d'un paramètre `name` reçu en entrée. Déploie-la et invoque-la via `aws lambda invoke`.

## Exercice 2 — Instance RDS

Décris (en commandes CLI) comment créer une instance RDS PostgreSQL `db.t3.micro`, puis explique comment ton application s'y connecterait (endpoint, port, identifiants) sans jamais exposer l'instance publiquement.

## Exercice 3 — Déploiement sur ECS/Fargate

En repartant de l'image Docker `mon-api` du mini-projet [`../../docker/README.md`](../../docker/README.md), décris les étapes pour la pousser vers ECR puis la faire tourner comme service ECS/Fargate.

## Exercice 4 — Alarme CloudWatch

Crée une alarme CloudWatch qui se déclenche si l'utilisation CPU moyenne d'une instance EC2 dépasse 80% pendant 10 minutes consécutives (deux périodes de 5 minutes).

## Exercice 5 — Zone Route 53

Décris (en commandes CLI ou en pseudo-config) comment créer une zone hébergée Route 53 pour un nom de domaine, et y ajouter un enregistrement qui pointe vers l'IP publique d'une instance EC2.
