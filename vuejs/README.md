# Vue.js

## 1. Introduction

Vue.js est un framework front-end **progressif** : on peut l'adopter composant par composant sur une page existante, ou construire une SPA complète avec son écosystème (Router, Pinia). Ce dossier couvre **Vue 3** avec la **Composition API** (le standard actuel) ; l'Options API historique est mentionnée pour la culture legacy mais pas développée. JS et TypeScript sont considérés acquis (voir [`../javascript/`](../javascript/) et [`../typescript/`](../typescript/)).

**À quoi sert-il ?**
- Construire des interfaces réactives déclaratives : le DOM se met à jour automatiquement quand l'état change, sans manipulation manuelle.
- Découper une UI en composants réutilisables, isolés et testables.
- Structurer une SPA (avec Vue Router + Pinia) ou enrichir des pages rendues côté serveur (PHP/Twig/Blade compris) avec des îlots interactifs.

**Où se situe-t-il dans une architecture web ?**
Couche présentation côté client. Consomme des API (REST/GraphQL) exposées par un backend (PHP, Node...). Pour du rendu serveur/SSR avec Vue, voir [`../nuxtjs/`](../nuxtjs/) — ce dossier-ci reste au niveau composant, sans SSR.

**Avantages**
- Courbe d'apprentissage douce (templates proches du HTML), très bonne documentation officielle.
- Réactivité fine et performante (Proxy-based depuis Vue 3), pas de VDOM diffing inutile grâce à la compilation des templates.
- Écosystème officiel cohérent (Router, Pinia, Vite) plutôt qu'un patchwork de libs tierces.

**Limites**
- Écosystème et bassin d'emploi plus petits que React dans certains marchés.
- La flexibilité template vs JSX peut sembler limitante pour de la logique de rendu très dynamique (bien que `<script setup>` + render functions comblent ce besoin).
- Moins de ressources tierces (composants UI, tutoriels) que React, quoique l'écart se réduit.

## 2. Prérequis

- JavaScript solide (closures, this, modules ES, destructuring, spread/rest) — voir [`../javascript/`](../javascript/).
- TypeScript de base recommandé (Vue 3 est écrit en TS et son tooling l'exploite pleinement) — voir [`../typescript/`](../typescript/).
- Node.js et npm installés (tooling Vite) — voir [`../nodejs/`](../nodejs/).
- Notions HTML/CSS pour le template et le style des composants.

## 3. Rappel des bases 🟢

### 01 - Créer un projet

```bash
npm create vue@latest
cd mon-projet && npm install && npm run dev
```

Le scaffolding officiel (Vite + Vue) propose TypeScript, Router, Pinia, ESLint en options dès la création — coche-les plutôt que de les ajouter après coup.

**Bonne pratique** : toujours partir de `npm create vue@latest` (ou Vite directement) plutôt que d'un boilerplate custom, pour bénéficier de la config officielle à jour.

### 02 - Anatomie d'un composant `.vue`

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Compteur : {{ count }}</button>
</template>

<style scoped>
button { padding: 0.5rem 1rem; }
</style>
```

Trois blocs : `<script setup>` (logique, sucre syntaxique de la Composition API), `<template>` (DOM déclaratif), `<style scoped>` (CSS isolé au composant via attributs générés automatiquement).

**Erreur fréquente** : oublier `scoped` et voir ses styles fuiter globalement dans l'app.

### 03 - Réactivité : `ref` et `reactive`

```js
import { ref, reactive } from 'vue'

const count = ref(0)        // valeur primitive réactive, accès via .value en JS
count.value++

const user = reactive({ name: 'Max', age: 30 })  // objet réactif, accès direct
user.age++
```

**Explication** — `ref` enveloppe n'importe quelle valeur dans un objet `{ value }` observé ; dans le `<template>`, Vue déballe automatiquement `.value` (pas besoin d'écrire `count.value` dans le HTML). `reactive` rend un objet réactif en profondeur mais **ne fonctionne que sur les objets/tableaux**, pas les primitives.

**Erreur fréquente** : déstructurer un objet `reactive` casse la réactivité (`const { name } = user` donne une copie figée). Utiliser `toRefs()` si besoin de déstructurer.

```js
import { toRefs } from 'vue'
const { name, age } = toRefs(user) // name et age restent réactifs
```

**Bonne pratique** : `ref` par défaut pour tout (cohérence, fonctionne pour primitives et objets), `reactive` seulement pour un état objet qui ne sera jamais déstructuré ni réassigné entièrement.

### 04 - Interpolation et directives

```vue
<template>
  <p>{{ message }}</p>
  <p v-if="isVisible">Visible seulement si isVisible est vrai</p>
  <p v-else>Sinon</p>
  <ul>
    <li v-for="item in items" :key="item.id">{{ item.label }}</li>
  </ul>
  <input :value="text" @input="text = $event.target.value" />
  <input v-model="text" /> <!-- équivalent, sucre syntaxique -->
</template>
```

- `{{ }}` : interpolation de texte.
- `v-if` / `v-else` / `v-show` : rendu conditionnel (`v-if` ajoute/retire du DOM, `v-show` bascule `display`, plus performant pour du toggle fréquent).
- `v-for` : boucle, **`:key` obligatoire et unique** pour que Vue réconcilie correctement le DOM.
- `:attr` (raccourci de `v-bind:attr`) : liaison d'attribut. `@event` (raccourci de `v-on:event`) : écoute d'événement.
- `v-model` : liaison bidirectionnelle, sucre pour `:value` + `@input`.

**Erreur fréquente** : utiliser l'index du tableau comme `:key` dans un `v-for` où la liste est réordonnée/filtrée — casse la réconciliation (état de composants mal réutilisé). Utiliser un identifiant stable de la donnée.

### 05 - Props et emits

```vue
<!-- ChildComponent.vue -->
<script setup>
const props = defineProps({
  title: { type: String, required: true },
  count: { type: Number, default: 0 }
})
const emit = defineEmits(['update'])
function increment() {
  emit('update', props.count + 1)
}
</script>
<template>
  <button @click="increment">{{ title }} : {{ count }}</button>
</template>
```

```vue
<!-- Parent -->
<ChildComponent :title="'Score'" :count="score" @update="score = $event" />
```

**Explication** — les données descendent par props (lecture seule côté enfant, jamais mutées directement), les événements remontent par `emit`. Flux de données **unidirectionnel** : c'est ce qui rend une app Vue prévisible à déboguer.

**Erreur fréquente** : muter une prop directement dans l'enfant (`props.count++`) — Vue lève un warning, car ça casse la source de vérité unique côté parent.

**Bonne pratique** : si l'enfant a besoin d'un état local dérivé d'une prop, le copier dans un `ref` local plutôt que de la muter.

## 4. Concepts intermédiaires 🟡

### Composition API en profondeur

```js
import { ref, computed, watch, watchEffect, onMounted, onUnmounted } from 'vue'

const price = ref(100)
const quantity = ref(2)

// computed : valeur dérivée, mise en cache, recalculée seulement si ses dépendances changent
const total = computed(() => price.value * quantity.value)

// watch : réagit explicitement à un changement de valeur précise
watch(quantity, (newVal, oldVal) => {
  console.log(`Quantité : ${oldVal} → ${newVal}`)
})

// watchEffect : ré-exécute automatiquement en trackant TOUTES les dépendances utilisées à l'intérieur
watchEffect(() => {
  console.log(`Total actuel : ${total.value}`)
})

onMounted(() => console.log('Composant monté, DOM disponible'))
onUnmounted(() => console.log('Nettoyage (listeners, timers...)'))
```

**Quand utiliser quoi** : `computed` pour une valeur dérivée pure (pas d'effet de bord) ; `watch` pour réagir à un changement précis avec accès à l'ancienne/nouvelle valeur (side effects ciblés : appel API, log) ; `watchEffect` pour du code qui doit simplement "rester synchronisé" avec ses dépendances sans lister explicitement lesquelles.

**Erreur fréquente** : mettre un effet de bord (appel réseau) dans un `computed` — un computed doit être pur, sinon le cache et les re-calculs deviennent imprévisibles.

### Composables (logique réutisable)

```js
// composables/useCounter.js
import { ref } from 'vue'

export function useCounter(initial = 0) {
  const count = ref(initial)
  const increment = () => count.value++
  const decrement = () => count.value--
  return { count, increment, decrement }
}
```

```vue
<script setup>
import { useCounter } from '@/composables/useCounter'
const { count, increment } = useCounter(10)
</script>
```

**Explication** — un composable est une simple fonction qui utilise les primitives de réactivité (`ref`, `computed`...) et peut être appelée dans n'importe quel composant : c'est le mécanisme d'extraction/réutilisation de logique de Vue 3 (équivalent des hooks React).

**Bonne pratique** : convention de nommage `useXxx`, un composable par fichier dans `composables/`, retourner un objet de refs/fonctions plutôt qu'un objet `reactive` (préserve la réactivité à la déstructuration côté appelant).

### Slots

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <header><slot name="header">Titre par défaut</slot></header>
    <main><slot /></main> <!-- slot par défaut -->
  </div>
</template>
```

```vue
<Card>
  <template #header>Mon titre custom</template>
  Contenu principal projeté dans le slot par défaut
</Card>
```

Les slots permettent à un composant parent d'injecter du contenu (voire du contenu paramétré via les *scoped slots*) dans la structure d'un composant enfant — essentiel pour des composants génériques (Modal, Card, Layout).

### Vue Router (bases)

```js
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: HomeView },
    { path: '/users/:id', component: UserView, props: true },
  ]
})

router.beforeEach((to, from) => {
  if (to.meta.requiresAuth && !isLoggedIn()) return '/login'
})
```

Navigation guards (`beforeEach`, `beforeEnter`) : centralisent la logique d'accès (auth, permissions) avant le rendu d'une route.

### Pinia (state management, bases)

```js
// stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({ name: '', isLoggedIn: false }),
  getters: {
    greeting: (state) => `Bonjour ${state.name}`
  },
  actions: {
    login(name) { this.name = name; this.isLoggedIn = true }
  }
})
```

```vue
<script setup>
import { useUserStore } from '@/stores/user'
const userStore = useUserStore()
</script>
<template>{{ userStore.greeting }}</template>
```

**Cas d'usage** : état partagé entre composants non liés par une relation parent-enfant directe (utilisateur connecté, panier, thème). Pour un état purement local à un composant, `ref`/`reactive` suffit — ne pas mettre tout dans Pinia par réflexe.

## 5. Concepts avancés 🟠🔴

### Performance

- **`v-once`** : rend une fois puis ne réactualise jamais ce fragment (contenu statique connu à l'avance).
- **`v-memo`** : mémorise un sous-arbre du template, ne re-rend que si les dépendances listées changent — utile pour de grosses listes (`v-for` avec beaucoup d'items).
- **`defineAsyncComponent`** : charge un composant lourd à la demande (code-splitting), combiné à `Suspense` pour gérer l'état de chargement.

```js
const HeavyChart = defineAsyncComponent(() => import('./HeavyChart.vue'))
```

**Erreur fréquente** : sur-utiliser `computed`/`watch` pour tout, y compris de la logique qui pourrait être une simple fonction pure appelée dans le template — chaque `computed`/`watch` a un coût de tracking de dépendances.

### Teleport et Suspense

```vue
<Teleport to="body">
  <div class="modal">Contenu rendu dans <body>, hors de la hiérarchie DOM du composant</div>
</Teleport>

<Suspense>
  <template #default><AsyncComponent /></template>
  <template #fallback>Chargement...</template>
</Suspense>
```

`Teleport` résout le problème classique des modals/tooltips piégés par un `overflow: hidden` ou un `z-index` d'un parent. `Suspense` gère l'état de chargement d'un arbre de composants async de façon déclarative.

### TypeScript avec Vue

```vue
<script setup lang="ts">
interface Props {
  title: string
  count?: number
}
const props = withDefaults(defineProps<Props>(), { count: 0 })

const emit = defineEmits<{
  update: [value: number]
}>()
</script>
```

Le typage des props/emits via des interfaces TS (plutôt que la syntaxe runtime `defineProps({...})`) donne l'autocomplétion et la vérification statique dans le template — à privilégier dans une base de code TS.

### Tests de composants

```js
import { mount } from '@vue/test-utils'
import Counter from './Counter.vue'

test('incrémente au clic', async () => {
  const wrapper = mount(Counter)
  await wrapper.find('button').trigger('click')
  expect(wrapper.text()).toContain('1')
})
```

Voir [`../testing/`](../testing/) pour la stratégie de test générale ; `@vue/test-utils` + Vitest est le combo standard pour tester des composants Vue en isolation.

### Architecture d'une grosse application

- Organiser par **feature/domaine** plutôt que par type technique quand l'app grossit (`features/users/{components,composables,store}` plutôt que `components/`, `composables/`, `store/` séparés globalement).
- Un store Pinia par domaine métier, pas un store géant.
- Composables pour toute logique partagée entre plusieurs composants (fetch de données, formulaires, permissions).

### SSR et hydration (aperçu)

Vue seul (via Vite) fait du rendu **client-only**. Pour du rendu serveur (SEO, temps de premier affichage), l'implémentation concrète (routing fichier, data fetching serveur, hydration) est traitée dans [`../nuxtjs/`](../nuxtjs/), qui encapsule ces mécanismes au-dessus de Vue.

## 6. Commandes / syntaxe à connaître

```bash
npm create vue@latest       # scaffolding d'un nouveau projet
npm install                 # installer les dépendances
npm run dev                 # serveur de dev (Vite)
npm run build                # build de production
npm run preview              # prévisualiser le build de prod en local
npm run test:unit            # tests unitaires (si configuré, ex. Vitest)
npx vue-tsc --noEmit          # vérification TypeScript des .vue
```

```js
// Directives essentielles
v-if / v-else-if / v-else
v-show
v-for (avec :key)
v-bind / :  (raccourci)
v-on / @   (raccourci)
v-model
v-slot / #  (raccourci)

// API réactivité
ref(), reactive(), computed(), watch(), watchEffect()
onMounted(), onUpdated(), onUnmounted()
```

## 7. Exercices

Voir [`exercices/niveau-1.md`](exercices/niveau-1.md), [`niveau-2.md`](exercices/niveau-2.md), [`niveau-3.md`](exercices/niveau-3.md). Corrections séparées dans [`solutions/`](solutions/).

## 8. Mini-projet

**Gestionnaire de tâches (todo-list) avec filtres et persistance locale**

- Composant `TodoApp` avec `ref`/`reactive` pour la liste de tâches.
- Ajouter, cocher, supprimer une tâche ; filtrer (toutes/actives/terminées) via un `computed`.
- Extraire la logique de gestion des tâches dans un composable `useTodos()`.
- Persister la liste dans `localStorage` via un `watch` sur la liste (profond, `{ deep: true }`).
- Bonus : découper en sous-composants (`TodoItem`, `TodoFilters`) avec props/emits, et un store Pinia si la todo-list doit être partagée entre plusieurs vues.

## Checklist

- [ ] Comprendre les fondamentaux (réactivité, template, props/emits)
- [ ] Savoir créer un projet Vue avec Vite
- [ ] Maîtriser la Composition API (`ref`, `computed`, `watch`, composables)
- [ ] Comprendre les concepts importants (slots, Vue Router, Pinia)
- [ ] Savoir debugger (Vue Devtools, warnings de réactivité)
- [ ] Connaître les bonnes pratiques (flux de données unidirectionnel, organisation par feature)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (performance, Teleport/Suspense, SSR en aperçu)

## 10. Ressources

- [Documentation officielle Vue.js](https://vuejs.org/guide/introduction.html) — à privilégier systématiquement, très complète et à jour.
- [Vue Router](https://router.vuejs.org/)
- [Pinia](https://pinia.vuejs.org/)
- [Vue Test Utils](https://test-utils.vuejs.org/)
- [roadmap.sh — Vue](https://roadmap.sh/vue) — vérifier qu'aucune notion majeure n'est manquante.
