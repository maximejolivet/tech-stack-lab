# Exercices GitHub — Niveau 1 (Bases)

## Exercice 1 — Fork vs clone

Explique en 2-3 phrases la différence entre forker un dépôt et le cloner, et donne un exemple de situation où l'on a besoin des deux à la fois.

## Exercice 2 — Ouvrir une Pull Request

Décris, étape par étape, le flux complet pour proposer une correction sur un projet dont tu n'es pas collaborateur direct (du fork jusqu'à l'ouverture de la PR).

## Exercice 3 — Lier une Issue à une PR

Écris la ligne à ajouter dans la description d'une Pull Request pour qu'elle ferme automatiquement l'issue numéro 27 à la fusion.

## Exercice 4 — Workflow CI minimal

Écris un workflow `.github/workflows/ci.yml` qui se déclenche sur chaque push et pull request, checkout le code, installe les dépendances avec `npm ci`, puis lance `npm test`.

## Exercice 5 — Branch protection

Décris deux règles de protection que tu activerais sur la branche `main` d'un projet en production, et explique pour chacune le risque qu'elle évite.
