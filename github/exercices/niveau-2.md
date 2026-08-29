# Exercices GitHub — Niveau 2 (Intermédiaire)

## Exercice 1 — Matrix build

Modifie le workflow de l'exercice précédent pour exécuter les tests sur trois versions de Node.js (18, 20, 22) en parallèle via une stratégie matrix.

## Exercice 2 — Utiliser un secret

Le workflow doit maintenant appeler une API externe nécessitant une clé `API_KEY`. Écris le step qui expose cette clé (stockée en secret de dépôt) comme variable d'environnement, sans jamais l'écrire en clair dans le YAML.

## Exercice 3 — CODEOWNERS

Écris un fichier `.github/CODEOWNERS` qui assigne automatiquement `@alice` comme relecteur obligatoire sur tout le dossier `src/api/`, et `@bob` sur `src/frontend/`.

## Exercice 4 — Template de Pull Request

Écris un `.github/pull_request_template.md` avec trois sections : "Description du changement", "Comment tester", et une checklist ("J'ai ajouté des tests", "J'ai mis à jour la documentation").

## Exercice 5 — Stratégies de fusion

Explique en 2-3 phrases la différence entre *squash and merge* et *rebase and merge*, et indique laquelle tu choisirais pour un projet où chaque commit sur `main` doit correspondre exactement à une PR — justifie.
