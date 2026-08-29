# Exercices CI/CD — Niveau 2 (Intermédiaire)

## Exercice 1 — Blue-green vs canary

Explique en 2-3 phrases la différence entre un déploiement blue-green et un déploiement canary, et donne un critère pour choisir l'un plutôt que l'autre selon la taille du trafic de l'application.

## Exercice 2 — Migration zero-downtime

Une table `users` doit recevoir une nouvelle colonne `email_verified` en `NOT NULL DEFAULT false`. L'application est déployée en rolling update (anciennes et nouvelles instances coexistent brièvement). Décris les étapes successives (sur plusieurs déploiements) pour appliquer ce changement sans downtime ni erreur sur les instances encore anciennes.

## Exercice 3 — Feature flag

Explique en 2-3 phrases comment un feature flag permet un "rollback" instantané d'une fonctionnalité sans revert de code ni redéploiement, et dans quel cas cette approche ne suffit pas (il faut un vrai rollback de code).

## Exercice 4 — Job de déploiement conditionnel

Écris le fragment de pipeline (GitHub Actions) qui déploie automatiquement vers `staging` sur chaque push sur `main`, mais uniquement après le succès du job `test`.

## Exercice 5 — Scan de sécurité en pipeline

Ajoute à la pipeline un job `security-scan` qui s'exécute en parallèle de `test` (pas après) et qui doit lui aussi passer avant que le job `deploy` ne se déclenche.
