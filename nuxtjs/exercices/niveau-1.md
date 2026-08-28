# Exercices — Niveau 1 (Bases)

## Exercice 1 — Premier projet
Crée un projet Nuxt (`npx nuxi init`), lance le serveur de dev, et vérifie que la page `/` s'affiche.

## Exercice 2 — Pages et routing par fichiers
Crée trois pages : `pages/index.vue`, `pages/about.vue`, `pages/contact.vue`, chacune affichant un titre différent. Vérifie que les trois routes fonctionnent sans configuration de routeur manuelle.

## Exercice 3 — Route dynamique
Crée `pages/users/[id].vue` qui affiche l'identifiant de l'URL (via `useRoute()`). Teste avec `/users/1` et `/users/42`.

## Exercice 4 — Layout commun
Crée un `layouts/default.vue` avec un header et un footer fixes, et vérifie qu'il s'applique automatiquement à toutes tes pages.

## Exercice 5 — Composant auto-importé
Crée `components/AppBadge.vue` (affiche un texte passé en prop) et utilise-le dans `pages/index.vue` sans écrire d'`import`.
