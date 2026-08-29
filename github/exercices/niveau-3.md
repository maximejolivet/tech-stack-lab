# Exercices GitHub — Niveau 3 (Avancé)

## Exercice 1 — Reusable workflow

Transforme le workflow de CI (niveau 1, exercice 4) en *reusable workflow* acceptant en entrée la version de Node.js à utiliser, puis écris un second workflow qui l'appelle avec `node-version: '20'`.

## Exercice 2 — Composite action

Décris la structure d'une composite action `setup-project` qui regrouperait le checkout, l'installation de Node.js et `npm ci` en un seul step réutilisable. Quels fichiers faut-il créer et où ?

## Exercice 3 — Environment protégé

Configure (en description, pas en YAML seul) un environment `production` qui exige l'approbation manuelle d'au moins un reviewer avant que le job de déploiement ne s'exécute. Écris le fragment de workflow qui référence cet environment.

## Exercice 4 — Path filters dans un monorepo

Dans un monorepo avec `packages/api/` et `packages/web/`, écris le déclencheur (`on:`) d'un workflow qui ne doit se lancer que lorsque des fichiers sous `packages/api/` changent.

## Exercice 5 — Semantic release (conception)

Explique en 3-4 phrases comment un outil de semantic release détermine automatiquement la prochaine version (majeure/mineure/patch) à partir des messages de commit, et pourquoi cela nécessite une convention de nommage de commit (ex. Conventional Commits) appliquée par l'équipe.
