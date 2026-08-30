# Exercices AWS — Niveau 3 (Avancé)

## Exercice 1 — Infrastructure as Code minimale

Écris un extrait Terraform qui provisionne un bucket S3 privé et une politique IAM qui l'autorise en lecture/écriture pour un rôle donné — sans passer par la console ni la CLI en one-shot.

## Exercice 2 — Auto-scaling et load balancer

Conçois (en description d'architecture, pas de code) un dispositif pour une API dont le trafic varie fortement selon l'heure de la journée : décris le rôle de l'Application Load Balancer, de l'Auto Scaling Group, et les métriques qui déclencheraient l'ajout/retrait d'instances.

## Exercice 3 — Choix d'instances et coûts

Une équipe fait tourner en continu 24/7 un service backend à charge stable, et un job de traitement par lots nocturne qui peut être interrompu et relancé sans problème. Pour chacun des deux cas, choisis entre instances **On-Demand**, **Reserved** et **Spot**, et justifie en 2-3 phrases.

## Exercice 4 — Revue de sécurité IAM

Voici une politique IAM attachée à un utilisateur applicatif :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": "*", "Resource": "*" }
  ]
}
```

Identifie le(s) problème(s) et réécris une politique correspondant au principe du moindre privilège pour un utilisateur qui doit seulement lire et écrire dans un bucket S3 nommé `mon-bucket`. Voir [`../../security/`](../../security/) pour le principe général.

## Exercice 5 — VPC public/privé

Décris (en description d'architecture) comment répartir les ressources d'une application à trois niveaux (load balancer, API, base de données) entre sous-réseaux publics et privés d'un VPC, et explique pourquoi la base de données ne doit jamais être placée dans le sous-réseau public.
