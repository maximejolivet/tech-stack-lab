# Exercices CI/CD — Niveau 1 (Bases)

## Exercice 1 — CI vs Continuous Delivery vs Continuous Deployment

Explique en une phrase chacun de ces trois termes, et donne un exemple concret de la différence entre Continuous Delivery et Continuous Deployment pour une même équipe.

## Exercice 2 — Pipeline build/test

Écris une pipeline (au format GitHub Actions) avec un job `build` qui installe les dépendances et build l'application, et un job `test` qui dépend de `build` et lance les tests.

## Exercice 3 — Fail fast

Une pipeline lance dans l'ordre : lint (10s), tests unitaires (30s), tests d'intégration (5min). Explique pourquoi cet ordre est préférable à l'ordre inverse, en te basant sur le principe de fail fast.

## Exercice 4 — Cache de dépendances

Ajoute une étape de cache à la pipeline de l'exercice 2 pour éviter de retélécharger les dépendances npm à chaque run, avec une clé de cache basée sur `package-lock.json`.

## Exercice 5 — Artefact unique

Explique en 2-3 phrases pourquoi une pipeline ne doit jamais reconstruire l'image Docker à l'étape de déploiement, et quel risque concret cela évite.
