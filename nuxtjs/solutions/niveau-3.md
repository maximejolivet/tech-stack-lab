# Solutions — Niveau 3

## Exercice 1
```ts
// server/api/articles.ts
export default defineEventHandler(async () => {
  await new Promise((r) => setTimeout(r, 2000))
  return [{ id: 1, titre: 'Article lent' }]
})
```
```ts
// nuxt.config.ts
routeRules: {
  '/articles': { swr: 3600 },
}
```
Premier chargement : ~2s (exécution complète). Rechargements suivants dans la fenêtre `swr` : réponse quasi instantanée car Nitro sert la version en cache tout en revalidant en arrière-plan (stale-while-revalidate) — c'est la différence observable.

## Exercice 2
```vue
<!-- pages/articles/[slug].vue -->
<script setup lang="ts">
const route = useRoute()
const { data: article } = await useFetch(`/api/articles/${route.params.slug}`)

useSeoMeta({
  title: () => article.value?.titre,
  description: () => article.value?.resume,
})
</script>
```
En SSR, `useSeoMeta` injecte les balises `<title>`/`<meta>` dans le HTML généré par le serveur : visibles immédiatement dans "Afficher le code source" (contrairement à une génération uniquement côté client après hydratation, invisible pour un crawler qui n'exécute pas JS).

## Exercice 3
Composant fautif : boucle de 5000 `<div>` réactifs montés directement sur `/`. Dans DevTools → Performance, le temps de "Scripting"/hydratation augmente nettement.

Corrections possibles :
- Rendre la liste conditionnelle (`v-if` sur une interaction utilisateur plutôt qu'au chargement).
- Découper via pagination ou virtualisation (n'afficher que les éléments visibles).
- Charger le composant en lazy (`<LazyMonComposant />`, préfixe automatique Nuxt pour les composants) pour ne l'hydrater qu'à la demande.

## Exercice 4
```ts
// server/api/meteo.ts
export default defineEventHandler(async () => {
  const config = useRuntimeConfig()
  const res = await $fetch('https://api.meteo.example/current', {
    query: { key: config.meteoApiKey },   // clé secrète, jamais exposée au client
  })
  return { temperature: res.temp, ville: res.city }
})
```
Commentaire attendu : appeler l'API météo directement depuis le composant Vue exposerait la clé API dans le bundle/le réseau côté client (visible dans l'onglet Network du navigateur) — la route serveur agit comme un proxy qui garde le secret côté serveur.

## Exercice 5
Proposition de répartition (fichier texte séparé attendu, exemple de contenu) :
- `/` , `/produits/**` (catalogue public) → **SSG ou SWR** : contenu public, peu volatile, SEO important → build statique ou cache revalidé.
- `/produits/[id]` (fiche produit avec stock en temps réel) → **SWR court** (ex: 60s) : compromis entre fraîcheur et performance.
- `/panier`, `/compte/**` (zone connectée, données propres à l'utilisateur) → **SSR classique ou SPA** : pas de SEO utile, données dynamiques par utilisateur, pas de cache partagé pertinent.
- `/admin/**` → **SPA (`ssr: false`)** : outil interne, pas de SEO, priorité à l'interactivité plutôt qu'au temps de premier rendu.
