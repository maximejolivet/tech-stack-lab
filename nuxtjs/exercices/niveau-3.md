# Exercices — Niveau 3 (Avancé)

## Exercice 1 — Cache SWR
Mets en place une route `server/api/articles.ts` simulant un appel lent (délai artificiel de 2s), consommée par une page en SSR. Ajoute une `routeRule` `swr` sur cette page et observe, avec plusieurs rechargements successifs, la différence de temps de réponse avant/après mise en cache.

## Exercice 2 — SEO dynamique
Sur une page de détail (`pages/articles/[slug].vue`), utilise `useSeoMeta()` pour générer dynamiquement titre et description à partir des données de l'article récupéré. Vérifie dans le HTML source (view-source) que les meta sont bien présentes côté serveur, pas seulement après hydratation.

## Exercice 3 — Diagnostic de performance d'hydratation
Ajoute volontairement un composant lourd (ex: boucle générant 5000 éléments avec de l'état réactif) chargé sur la page d'accueil. Mesure l'impact sur le temps d'hydratation (DevTools Performance), puis corrige en le rendant conditionnel ou en le découpant.

## Exercice 4 — BFF léger vers une API externe
Crée une route `server/api/meteo.ts` qui appelle une API météo publique côté serveur (masquant une éventuelle clé API) et retourne un format simplifié au client. Explique en commentaire pourquoi cet appel ne doit pas être fait directement depuis le composant Vue.

## Exercice 5 — Choix d'architecture
Pour un projet fictif "plateforme e-commerce avec catalogue public + espace client connecté" : propose (par écrit, dans un fichier séparé) une répartition des modes de rendu par route (`routeRules`) et justifie chaque choix (SSR/SSG/SPA/SWR).
