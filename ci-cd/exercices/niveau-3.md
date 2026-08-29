# Exercices CI/CD — Niveau 3 (Avancé)

## Exercice 1 — Rollback automatique piloté par métriques

Décris la logique (en pseudo-étapes, pas nécessairement en YAML complet) d'une pipeline qui déploie une version en canary sur 10% du trafic, surveille le taux d'erreur pendant 5 minutes, et déclenche automatiquement un rollback si ce taux dépasse 2%.

## Exercice 2 — Approbation manuelle avant production

Écris le fragment de pipeline qui déploie automatiquement vers `staging`, puis exige une approbation manuelle (environment protégé) avant de déployer vers `production`.

## Exercice 3 — Idempotence

Explique en 2-3 phrases ce que signifie "une pipeline de déploiement idempotente", et donne un exemple concret de déploiement qui ne le serait PAS (effet de bord destructeur si rejoué après un échec partiel).

## Exercice 4 — Infrastructure as Code dans la pipeline

Décris à quel moment, dans une pipeline complète (build/test/package/deploy), un changement d'infrastructure (ex. un fichier Terraform modifiant une base de données managée) devrait être appliqué par rapport au déploiement du code applicatif, et pourquoi cet ordre compte.

## Exercice 5 — Observabilité de la pipeline

Propose trois métriques à suivre sur la pipeline elle-même (pas sur l'application) qui permettraient de détecter qu'elle devient une source de friction pour l'équipe, et explique pour chacune ce qu'un mauvais chiffre indiquerait.
