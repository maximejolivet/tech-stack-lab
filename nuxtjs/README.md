# Nuxt.js

## 1. Introduction

Nuxt est un **méta-framework full-stack** construit sur Vue.js : il ajoute par-dessus Vue tout ce qu'une vraie application en production a besoin (rendu serveur, routing par fichiers, API intégrée, conventions de structure) sans avoir à assembler soi-même Vue Router, Vite, un serveur SSR, etc. Les bases de Vue (Composition API, réactivité, composants, Pinia) sont supposées acquises — voir [`../vuejs/`](../vuejs/) — ce dossier se concentre uniquement sur ce que **Nuxt ajoute**.

**À quoi sert-il ?**
- Éviter de reconfigurer manuellement SSR, routing, build, code-splitting à chaque projet Vue.
- Fournir des conventions (structure de dossiers) qui remplacent la configuration répétitive.
- Unifier front et back léger dans un seul projet via le dossier `server/`.

**Où se situe-t-il dans une architecture web ?**
Côté front pour le rendu (SSR/SSG/SPA), et potentiellement côté "backend for frontend" léger (API routes internes, proxy vers une vraie API PHP/Java). Il ne remplace pas un backend métier complet (Symfony/Laravel/Spring Boot) mais peut l'appeler ou exposer de petites routes utilitaires.

**Avantages**
- SSR/SSG "out of the box", excellent pour le SEO et les performances de premier chargement.
- Auto-import : moins de boilerplate (pas d'imports manuels de composants/composables).
- Écosystème de modules riche (Nuxt Content, Nuxt Image, Nuxt UI...).

**Limites**
- Couche d'abstraction supplémentaire : plus de "magie" (auto-imports, conventions implicites) qu'un projet Vite+Vue nu, courbe d'apprentissage des spécificités Nuxt.
- Le SSR ajoute de la complexité (hydratation, code qui ne doit tourner que côté client ou que côté serveur).
- Changements de version majeurs (Nuxt 2 → 3) ont cassé beaucoup de choses : vérifier la maintenance des modules tiers.

## 2. Prérequis

- Vue.js 3 et la Composition API maîtrisés — voir [`../vuejs/`](../vuejs/) si besoin de réviser avant.
- JavaScript/TypeScript modernes (voir [`../javascript/`](../javascript/), [`../typescript/`](../typescript/)).
- Notions de base HTTP (méthodes, statuts, JSON) utiles pour la partie `server/`.

## 3. Rappel des bases 🟢

### 01 - Pourquoi Nuxt plutôt que Vue seul

**Explication** — Un projet Vue+Vite "nu" ne fait ni SSR, ni routing par fichiers, ni auto-import : il faut installer et configurer Vue Router, un serveur Node pour le SSR, etc. Nuxt fournit tout ça par convention.

**Cas d'usage** : site public avec besoin de SEO (SSR/SSG), application avec beaucoup de pages/routes, projet où la vitesse de mise en place prime.

**Erreur fréquente** : utiliser Nuxt pour un dashboard interne 100% SPA sans besoin de SEO — dans ce cas, Vue+Vite seul est plus simple et suffisant (Nuxt en mode SPA reste possible mais apporte moins de valeur).

**Bonne pratique** : choisir Nuxt quand le SEO, le temps de premier rendu (SSR) ou les conventions de structure apportent une vraie valeur — pas par défaut systématique.

### 02 - Créer un projet Nuxt

```bash
npx nuxi@latest init mon-projet
cd mon-projet
npm install
npm run dev
```

**Bonne pratique** : figer la version de Nuxt dans `package.json` (`~3.x.x`) plutôt que `^3.x.x` en production, pour éviter des montées de version majeures non maîtrisées.

### 03 - Structure de dossiers

```text
mon-projet/
├── pages/          # routing basé sur les fichiers
├── components/     # composants Vue, auto-importés
├── layouts/        # gabarits de page (default.vue, etc.)
├── composables/     # fonctions composables, auto-importées
├── server/          # API routes + logique serveur (Nitro)
├── public/          # fichiers statiques servis tels quels
├── assets/          # assets à transformer par le build (CSS, images)
└── nuxt.config.ts
```

**Erreur fréquente** : mettre des fichiers statiques (favicon, robots.txt) dans `assets/` au lieu de `public/` — `assets/` passe par le build, `public/` est servi tel quel à la racine.

**Bonne pratique** : respecter strictement ces conventions de nommage de dossiers — c'est ce qui permet à Nuxt de générer le routing et les auto-imports automatiquement.

### 04 - Routing basé sur les fichiers (file-based routing)

**Explication** — Chaque fichier `.vue` dans `pages/` devient une route, sans configuration de routeur manuelle.

```text
pages/
├── index.vue           →  /
├── about.vue            →  /about
├── users/
│   ├── index.vue        →  /users
│   └── [id].vue          →  /users/:id   (paramètre dynamique)
└── users/[id]/edit.vue    →  /users/:id/edit
```

```vue
<!-- pages/users/[id].vue -->
<script setup lang="ts">
const route = useRoute()
const userId = route.params.id
</script>
```

**Cas d'usage** : toute app avec plusieurs pages — c'est le cas d'usage central de Nuxt.

**Erreur fréquente** : oublier que `[id].vue` et `[id]/index.vue` ne sont pas strictement équivalents pour les routes imbriquées (nested routes via `<NuxtPage>`).

**Bonne pratique** : garder une arborescence `pages/` qui reflète directement l'arborescence d'URL voulue — pas de logique de routing cachée ailleurs.

### 05 - Layouts

```vue
<!-- layouts/default.vue -->
<template>
  <div>
    <AppHeader />
    <slot />
    <AppFooter />
  </div>
</template>
```

```vue
<!-- pages/about.vue -->
<script setup lang="ts">
definePageMeta({ layout: 'custom' })
</script>
```

**Bonne pratique** : un layout par grande zone de l'app (public vs admin par exemple), pas un layout par page.

### 06 - Auto-import (composants et composables)

**Explication** — Tout composant dans `components/` et toute fonction exportée depuis `composables/` sont utilisables sans `import` explicite.

```vue
<!-- components/AppButton.vue existe → utilisable direct -->
<template>
  <AppButton>Cliquer</AppButton>
</template>
```

**Erreur fréquente** : créer deux composants avec le même nom de fichier dans des sous-dossiers différents de `components/` — Nuxt préfixe alors le nom (`components/base/Button.vue` → `<BaseButton>`), source de confusion si non anticipé.

**Bonne pratique** : nommer les fichiers composants de façon unique et explicite pour éviter les préfixes automatiques surprenants ; en cas de doute, vérifier avec `.nuxt/components.d.ts` généré.

### 07 - `nuxt.config.ts`

```ts
export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: ['@pinia/nuxt'],
  runtimeConfig: {
    apiSecret: '',           // uniquement côté serveur
    public: {
      apiBase: '/api',        // exposé aussi côté client
    },
  },
})
```

**Erreur fréquente** : mettre un secret dans `runtimeConfig.public` — tout ce qui est dans `public` est envoyé au navigateur.

**Bonne pratique** : séparer clairement ce qui est secret (racine de `runtimeConfig`) de ce qui est public, et charger les vraies valeurs depuis des variables d'environnement (`NUXT_API_SECRET`, `NUXT_PUBLIC_API_BASE`).

## 4. Concepts intermédiaires 🟡

**Modes de rendu** — Nuxt supporte plusieurs stratégies configurables : SSR (rendu serveur à chaque requête, par défaut), SSG (`nuxt generate`, pages pré-générées en HTML statique au build), SPA (`ssr: false`, rendu 100% client), et le rendu hybride par route via `routeRules` dans `nuxt.config.ts` (ex: page marketing en SSG, dashboard en SPA, dans le même projet). Choisir le mode par route plutôt que globalement dès que les besoins diffèrent (page publique vs zone connectée).

**Data fetching** — `useFetch()` et `useAsyncData()` sont conçus pour le SSR : ils exécutent la requête côté serveur au premier rendu puis réutilisent le résultat côté client (pas de double appel), contrairement à un `fetch()` classique dans `onMounted` qui ne s'exécuterait que côté client (mauvais pour le SEO et pour le temps de premier affichage).

```vue
<script setup lang="ts">
const { data: users, pending, error, refresh } = await useFetch('/api/users')
</script>
```

**Erreur fréquente** : appeler `fetch()` natif directement dans `<script setup>` pour du data fetching de page — ça casse le partage serveur/client et duplique l'appel réseau.

**Le dossier `server/`** — Nitro (le moteur serveur de Nuxt) permet d'écrire des routes API directement dans le projet, sans backend séparé pour des besoins simples :

```ts
// server/api/hello.ts
export default defineEventHandler((event) => {
  return { message: 'Hello' }
})
// accessible en GET /api/hello
```

**Cas d'usage** : proxy vers une vraie API (masquer une clé secrète), agrégation de plusieurs appels, endpoints utilitaires légers. Pour une vraie logique métier, ça reste préférable de s'appuyer sur le backend principal (Symfony/Laravel/Spring Boot) plutôt que de tout mettre dans `server/`.

**Gestion d'état** — `useState()` pour un état partagé simple et SSR-safe (évite les fuites d'état entre requêtes, contrairement à une variable module-level classique) ; Pinia (via le module `@pinia/nuxt`) pour un state management plus structuré, comme en Vue pur.

**Middleware de route** — fonctions exécutées avant la navigation vers une page (`middleware/auth.ts` + `definePageMeta({ middleware: 'auth' })`), utiles pour les redirections d'authentification.

**Modules Nuxt** — l'écosystème de modules (`@nuxt/image`, `@pinia/nuxt`, `@nuxtjs/tailwindcss`...) s'installe via `modules` dans `nuxt.config.ts` et étend Nuxt sans configuration manuelle lourde. Vérifier la compatibilité Nuxt 3 et la maintenance active avant d'en ajouter un.

**Plugins** — code exécuté à l'initialisation de l'app (`plugins/`), utile pour enregistrer une librairie tierce globalement (ex: un client HTTP configuré).

## 5. Concepts avancés 🟠🔴

**Cache et ISR** — Via `routeRules`, Nuxt/Nitro permet du cache par route et de l'Incremental Static Regeneration (régénération d'une page statique après expiration d'un délai, sans rebuild complet) : utile pour du contenu qui change peu mais pas jamais (fiches produit, articles).

```ts
routeRules: {
  '/produits/**': { swr: 3600 },   // stale-while-revalidate, 1h
  '/admin/**': { ssr: false },     // SPA pur pour la zone admin
}
```

**Performance** — L'hydratation (le moment où Vue "reprend la main" côté client sur le HTML déjà rendu par le serveur) est un point critique : trop de composants interactifs hydratés inutilement ralentit le chargement perçu. Nuxt fait du code-splitting automatique par route. `<NuxtImg>` (module `@nuxt/image`) gère le redimensionnement/format d'image automatiquement — à préférer à une balise `<img>` brute pour des images de contenu.

**SEO** — `useSeoMeta()` et `useHead()` permettent de définir titre/meta/Open Graph par page, essentiel puisque le SSR rend ces meta visibles aux crawlers dès le premier chargement (contrairement à une SPA pure où le SEO est plus difficile). Génération de sitemap via un module dédié (`@nuxtjs/sitemap`).

**Déploiement** — Nitro compile vers différentes cibles ("presets") : serveur Node classique, ou plateformes serverless/edge (Vercel, Netlify, Cloudflare Workers...) sans changement de code applicatif, juste une configuration de preset. Comprendre la cible de déploiement en amont influence certains choix (taille du bundle serveur, cold starts en edge/serverless).

**Architecture à grande échelle** — Sur un gros projet Nuxt : séparer clairement les composables "métier" (logique réutilisable) des composants "présentation", éviter de surcharger `server/` avec de la logique métier complexe (rester une fine couche BFF), monitorer le temps de rendu SSR (un composable lent bloque le TTFB de toutes les pages qui l'utilisent).

## 6. Commandes / syntaxe à connaître

```bash
npx nuxi@latest init <projet>   # créer un nouveau projet
npm run dev                      # serveur de dev
npm run build                    # build de production (SSR)
npm run generate                 # build statique (SSG)
npm run preview                  # prévisualiser le build en local
npx nuxi add page nom            # générer une page
npx nuxi add component Nom       # générer un composant
npx nuxi typecheck                # vérification TypeScript
```

## 7. Exercices

Énoncés dans [`exercices/`](./exercices/) : [niveau 1](./exercices/niveau-1.md), [niveau 2](./exercices/niveau-2.md), [niveau 3](./exercices/niveau-3.md). Solutions séparées dans [`solutions/`](./solutions/) — à consulter seulement après avoir essayé.

## 8. Mini-projet

**Blog technique avec fiches articles** — Créer une petite app Nuxt : liste d'articles sur `/articles` (récupérés via `useFetch` depuis une route `server/api/articles.ts` qui retourne un tableau statique ou lit un JSON), page de détail `/articles/[slug].vue` avec `useSeoMeta()` pour le titre/description dynamiques, layout commun avec header/footer, et une `routeRule` de cache (`swr`) sur les pages articles. Bonus : ajouter Pinia pour un compteur de "vues" en mémoire par article.

## Checklist

- [ ] Comprendre les fondamentaux
- [ ] Savoir créer un projet
- [ ] Maîtriser la syntaxe principale
- [ ] Comprendre les concepts importants
- [ ] Savoir debugger
- [ ] Connaître les bonnes pratiques
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées

## 10. Ressources

- [Documentation officielle Nuxt](https://nuxt.com/docs)
- [Nitro (moteur serveur de Nuxt)](https://nitro.unjs.io/)
- [roadmap.sh — Vue](https://roadmap.sh/vue) (pas de roadmap.sh Nuxt dédiée à ce jour)
