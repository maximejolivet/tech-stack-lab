# Exercices AWS — Niveau 1 (Bases)

## Exercice 1 — Utilisateur IAM restreint

Crée un utilisateur IAM `deploy-bot` (sans utiliser le compte root pour les opérations courantes), avec une politique qui l'autorise **seulement** à lire et écrire dans un bucket S3 précis — pas d'accès administrateur global.

## Exercice 2 — Bucket S3 privé

Crée un bucket S3 avec un nom unique, uploade un fichier texte dedans, puis télécharge-le à nouveau sous un autre nom pour vérifier le contenu. Vérifie que le bucket reste privé par défaut (pas d'accès public).

## Exercice 3 — Instance EC2 et security group

Décris (en commandes CLI) comment lancer une instance EC2 `t3.micro`, avec un security group qui n'autorise que le port 22 (SSH) depuis une plage d'IP précise et le port 80 (HTTP) depuis n'importe où.

## Exercice 4 — Configuration de l'AWS CLI

Exécute `aws configure` et explique où sont stockés les identifiants ainsi créés sur ta machine. Explique pourquoi il ne faut jamais committer ce fichier dans un dépôt Git.

## Exercice 5 — Tags de ressources

Ajoute les tags `Environment=staging` et `Project=mon-api` à une instance EC2 (ou tout autre ressource de ton choix), puis liste les ressources filtrées par ce tag `Project`.
