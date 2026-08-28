# Exercices — Niveau 2 (Intermédiaire)

## Exercice 1 — API route interne
Crée `server/api/produits.ts` qui retourne un tableau JSON de 3 produits statiques (id, nom, prix). Depuis `pages/produits.vue`, récupère et affiche cette liste avec `useFetch('/api/produits')`.

## Exercice 2 — État partagé avec `useState`
Crée un composable `useCompteur()` basé sur `useState` qui expose un compteur partagé et une fonction pour l'incrémenter. Utilise-le dans deux composants différents affichés sur la même page et vérifie qu'ils partagent bien le même état.

## Exercice 3 — Middleware d'authentification simulé
Crée un middleware `middleware/auth.ts` qui redirige vers `/login` si une variable (simulée, ex: état global `isLoggedIn`) est fausse. Applique-le à une page `pages/dashboard.vue`.

## Exercice 4 — Mode de rendu par route
Configure `routeRules` dans `nuxt.config.ts` pour qu'une page (ex: `/admin/**`) soit rendue en SPA pur (`ssr: false`) pendant que le reste du site reste en SSR. Vérifie la différence en inspectant le HTML source retourné par le serveur.

## Exercice 5 — Runtime config
Ajoute une variable `apiBase` dans `runtimeConfig.public`, alimentée par une variable d'environnement `NUXT_PUBLIC_API_BASE`, et utilise-la dans un appel `useFetch`.
