# Svelte

## 1. Introduction

Svelte n'est pas un framework qui s'exécute dans le navigateur comme Vue ou React : c'est un **compilateur**. Au build, il transforme chaque composant `.svelte` en JavaScript vanilla optimisé qui manipule directement le DOM, plutôt que d'embarquer un runtime de diffing (Virtual DOM) chargé d'interpréter l'état à chaque changement. Ce dossier couvre **Svelte 5** (API des *runes* — `$state`, `$derived`, `$effect`, `$props` — le standard actuel) ainsi que **SvelteKit**, son méta-framework officiel pour le routing et le SSR. Contrairement à [`../vuejs/`](../vuejs/) et [`../nuxtjs/`](../nuxtjs/) qui sont deux dossiers séparés (framework puis méta-framework), Svelte et SvelteKit sont traités ensemble ici, sur le modèle combiné déjà utilisé pour [`../flutter/`](../flutter/) (Dart + Flutter) et [`../swift/`](../swift/) (Swift + SwiftUI) — SvelteKit étant maintenu par la même équipe et constituant le point d'entrée standard pour tout nouveau projet Svelte, pas une couche optionnelle tierce comme Nuxt l'est pour Vue.

**À quoi sert-il ?**
- Construire des interfaces réactives déclaratives avec un bundle final nettement plus léger qu'un équivalent React/Vue, puisqu'aucun runtime de framework n'est envoyé au navigateur.
- Structurer une application complète (routing, SSR/SSG, API routes) via SvelteKit, sur le même principe que Nuxt/Next mais avec des conventions plus minimalistes.

**Où se situe-t-il dans une architecture web ?**
Côté client par défaut pour un composant Svelte isolé ; SvelteKit ajoute une couche serveur (SSR, SSG, endpoints API) comparable à ce que Nuxt apporte à Vue — voir [`../nuxtjs/`](../nuxtjs/) pour les concepts partagés (routing par fichiers, data fetching serveur, déploiement edge/serverless).

**Avantages**
- Bundle final beaucoup plus petit à fonctionnalité équivalente : pas de runtime de framework à télécharger, seul le code strictement nécessaire au composant est généré.
- Moins de boilerplate : la réactivité est une propriété du langage compilé (`$state`) plutôt qu'une API à appeler (`useState`, `ref`) — le code source ressemble à du JS/HTML simple.
- Transitions et animations intégrées nativement au langage (`transition:`, `animate:`), sans dépendance tierce pour des cas d'usage courants.

**Limites**
- Écosystème et bassin d'emploi nettement plus restreints que React ou Vue — moins de composants tiers prêts à l'emploi, moins d'offres d'embauche.
- Étant plus récent (les runes de Svelte 5 datent de 2024), moins de retours d'expérience à grande échelle disponibles que sur React/Vue/Angular.
- Le modèle de compilation rend certains outils génériques (certains linters, certaines intégrations IDE) moins matures que sur l'écosystème JSX.

## 2. Prérequis

- JavaScript solide (closures, modules ES, destructuring, spread/rest) — voir [`../javascript/`](../javascript/).
- TypeScript de base recommandé (SvelteKit s'utilise couramment avec, et le tooling l'exploite bien) — voir [`../typescript/`](../typescript/).
- Node.js et npm installés (tooling Vite, sur lequel Svelte s'appuie) — voir [`../nodejs/`](../nodejs/).
- Avoir déjà pratiqué un autre framework réactif (Vue ou React, voir [`../vuejs/`](../vuejs/) et [`../react/`](../react/)) rend les comparaisons de ce dossier beaucoup plus parlantes — les concepts génériques (composant, props, rendu conditionnel) ne sont pas ré-expliqués depuis zéro ici.

## 3. Rappel des bases 🟢

### 01 - Créer un projet

**Explication** — Le CLI officiel scaffold un projet SvelteKit complet (routing, SSR, config Vite) en une commande — pas de projet "Svelte seul" recommandé pour un usage réel, SvelteKit est le point d'entrée standard.

```bash
npx sv create mon-projet
cd mon-projet && npm install && npm run dev
```

**Bonne pratique** : accepter TypeScript et ESLint proposés lors du scaffolding plutôt que de les ajouter après coup — la config officielle reste à jour avec les nouvelles versions de Svelte.

### 02 - Anatomie d'un fichier `.svelte`

**Explication** — Comme un composant Vue, un fichier `.svelte` regroupe script, markup et style en un seul fichier — mais le markup est du **HTML directement enrichi**, pas un langage de template séparé compilé à part.

```svelte
<script>
  let count = $state(0);
</script>

<button onclick={() => count++}>Compteur : {count}</button>

<style>
  button { padding: 0.5rem 1rem; }
</style>
```

**Différence clé avec Vue/React** : `{count}` dans le markup est directement une expression JS interpolée, `onclick` est un attribut HTML standard (pas de préfixe `@` ni `on` comme en Vue/React) — le compilateur reconnaît nativement que c'est un gestionnaire d'événement.

**Erreur fréquente** : chercher un bloc `<template>` séparé comme en Vue — il n'existe pas, tout ce qui n'est ni `<script>` ni `<style>` est directement le markup rendu.

### 03 - Réactivité avec `$state`

**Explication** — `$state()` marque une variable comme réactive : toute réassignation (ou mutation d'un tableau/objet marqué `$state`) déclenche automatiquement une mise à jour du DOM concerné, sans `.value` à dérérerencer comme le `ref` de Vue.

```svelte
<script>
  let count = $state(0);
  let user = $state({ name: 'Max', age: 30 });

  function increment() {
    count++;              // réassignation directe, pas de .value
    user.age++;            // mutation directe d'un objet $state, aussi réactive
  }
</script>
```

**Cas d'usage** : `$state` remplace à la fois `ref` et `reactive` de Vue — un seul mécanisme, quel que soit le type de la valeur (primitive, objet, tableau).

**Bonne pratique** : ne pas confondre avec l'ancienne syntaxe Svelte 4 (`let count = 0` réactif "par magie" sans rune, encore valable pour la rétrocompatibilité mais plus l'idiome recommandé) — les runes rendent la réactivité explicite et sont désormais le standard.

### 04 - Rendu conditionnel

**Explication** — Blocs `{#if}/{:else if}/{:else}` directement dans le markup, fermés par `{/if}` — syntaxe de contrôle propre à Svelte plutôt que des directives sur des balises (`v-if`) ou des expressions JS (`{cond && <X/>}`).

```svelte
<script>
  let isLoggedIn = $state(false);
</script>

{#if isLoggedIn}
  <Dashboard />
{:else}
  <LoginForm />
{/if}
```

**Erreur fréquente** : oublier la balise fermante `{/if}` — contrairement au JSX (une expression JS classique), ces blocs ont une syntaxe de fermeture explicite proche du Twig/Blade déjà rencontrés côté PHP.

### 05 - Boucles avec `{#each}`

**Explication** — `{#each liste as item (item.id)}` itère sur un tableau ; la clé entre parenthèses (équivalent du `:key` Vue ou de la prop `key` React) est **optionnelle syntaxiquement mais fortement recommandée** dès que la liste peut être réordonnée.

```svelte
<script>
  let tasks = $state([
    { id: 1, label: 'Réviser Svelte', done: false },
  ]);
</script>

<ul>
  {#each tasks as task (task.id)}
    <li>{task.label} — {task.done ? '✅' : '⏳'}</li>
  {/each}
</ul>
```

**Erreur fréquente** : omettre la clé `(task.id)` sur une liste filtrable/réordonnable — Svelte réutilise alors les nœuds DOM par position plutôt que par identité, provoquant les mêmes bugs d'état mal associé qu'un `:key` sur index en Vue/React.

### 06 - Gestion des événements

**Explication** — Les gestionnaires sont des attributs HTML natifs (`onclick`, `oninput`, `onsubmit`) recevant directement une fonction — pas de directive dédiée, pas de préfixe framework.

```svelte
<script>
  function handleSubmit(e) {
    e.preventDefault();
    console.log('soumis');
  }
</script>

<form onsubmit={handleSubmit}>
  <button type="submit">Envoyer</button>
</form>
```

**Bonne pratique** : passer la référence de fonction directement (`onclick={handleClick}`), une arrow function inline seulement quand un argument doit être capturé (`onclick={() => remove(item.id)}`) — même principe qu'en React.

### 07 - Props avec `$props()`

**Explication** — Un composant déclare ses props via la destructuration de `$props()`, avec valeurs par défaut directement en syntaxe JS standard — pas d'API de déclaration séparée (`defineProps` en Vue, typage à part en React).

```svelte
<!-- Alert.svelte -->
<script>
  let { message, type = 'info', onClose } = $props();
</script>

<div class={['alert', type]}>
  {message}
  <button onclick={onClose}>×</button>
</div>
```

```svelte
<!-- Parent -->
<Alert message="Sauvegardé avec succès" type="success" onClose={() => visible = false} />
```

**Explication** — Contrairement à Vue (props + `emit` séparés), Svelte 5 fait remonter les événements enfant → parent en passant simplement une **fonction en prop** (`onClose`), appelée directement par l'enfant — un modèle plus proche des callbacks React que du système d'événements Vue.

**Erreur fréquente** : réassigner directement une prop déstructurée (`message = 'autre'`) en pensant modifier l'affichage local sans effet de bord — Svelte avertit que ça casse le flux de données descendant, comme muter une prop React/Vue.

### 08 - Binding bidirectionnel avec `bind:value`

**Explication** — `bind:value` synchronise directement un champ de formulaire avec une variable `$state`, dans les deux sens — équivalent direct du `v-model` de Vue.

```svelte
<script>
  let text = $state('');
</script>

<input bind:value={text} />
<p>Vous avez tapé : {text} ({text.length} caractères)</p>
```

**Cas d'usage** : `bind:value` reste un sucre syntaxique pour `value={text} oninput={e => text = e.target.value}` — utile de connaître l'équivalent explicite pour les cas où une transformation de la valeur est nécessaire avant stockage.

## 4. Concepts intermédiaires 🟡

- **`$derived` (valeurs calculées)** : dérive une valeur à partir d'autres runes `$state`, recalculée automatiquement et seulement quand une dépendance change — équivalent direct du `computed` de Vue ou d'un `useMemo` React appliqué systématiquement.

```svelte
<script>
  let price = $state(100);
  let quantity = $state(2);
  let total = $derived(price * quantity);   // pas de fonction à appeler, juste une expression
</script>
```

- **`$effect` (effets de bord)** : exécute du code en réaction à un changement de state observé à l'intérieur, avec nettoyage possible via une fonction de retour — équivalent du `watchEffect` de Vue ou du `useEffect` de React.

```svelte
<script>
  $effect(() => {
    console.log(`Total actuel : ${total}`);
    return () => console.log('nettoyage avant le prochain effet');
  });
</script>
```

**Erreur fréquente** : utiliser `$effect` pour dériver une valeur (ce que `$derived` fait déjà, plus simplement et sans effet de bord) — même piège que sur-utiliser `useEffect` en React pour du calcul pur.

- **Composition et contenu projeté (`{#snippet}` / children)** : en Svelte 5, un composant reçoit du contenu à projeter via `children` (une prop spéciale, comme en React) ou des `{#snippet}` nommés pour plusieurs zones de contenu réutilisables — équivalent fonctionnel des slots nommés de Vue.

```svelte
<!-- Card.svelte -->
<script>
  let { header, children } = $props();
</script>
<div class="card">
  <header>{@render header?.()}</header>
  <main>{@render children()}</main>
</div>
```

```svelte
<Card>
  {#snippet header()}<h2>Mon titre</h2>{/snippet}
  Contenu principal projeté dans les children
</Card>
```

- **Stores (`writable`/`readable`)** : pour un état partagé entre composants sans relation parent-enfant directe, un store expose une valeur observable en dehors du cycle de vie d'un composant — équivalent conceptuel de Pinia (voir [`../vuejs/`](../vuejs/)), bien que plus minimaliste (pas de getters/actions structurés par défaut, juste un conteneur réactif).

```js
// stores/cart.js
import { writable } from 'svelte/store';

export const cart = writable([]);
export function addItem(item) {
  cart.update(items => [...items, item]);
}
```

```svelte
<script>
  import { cart } from './stores/cart.js';
</script>
<p>{$cart.length} articles</p> <!-- le préfixe $ auto-souscrit au store dans le markup -->
```

- **Routing par fichiers avec SvelteKit** : chaque dossier de `src/routes/` devient une route, `+page.svelte` étant le composant affiché — même principe que le routing par fichiers de Nuxt (voir [`../nuxtjs/`](../nuxtjs/)), avec `+layout.svelte` pour un gabarit partagé entre plusieurs routes.

```text
src/routes/
├── +layout.svelte        # layout partagé (header/footer)
├── +page.svelte           # →  /
└── users/
    └── [id]/
        └── +page.svelte    # →  /users/:id
```

- **Chargement de données avec `load`** : une fonction `load` exportée depuis `+page.js` (client/serveur) ou `+page.server.js` (serveur uniquement) prépare les données d'une page avant son rendu — équivalent du `useFetch`/`useAsyncData` de Nuxt, mais résolu par SvelteKit avant même de monter le composant.

```js
// +page.js
export async function load({ fetch, params }) {
  const res = await fetch(`/api/users/${params.id}`);
  return { user: await res.json() };
}
```

```svelte
<!-- +page.svelte -->
<script>
  let { data } = $props();   // "data" est injecté automatiquement par SvelteKit
</script>
<p>{data.user.name}</p>
```

## 5. Concepts avancés 🟠🔴

- **Le modèle de compilation, sans Virtual DOM** : là où React/Vue diffent un arbre virtuel à chaque changement d'état pour déterminer les mises à jour DOM nécessaires (voir [`../react/`](../react/), section reconciliation), Svelte génère au **build** des instructions de mise à jour DOM chirurgicales et spécifiques à chaque composant (`if (changed.count) text.data = count`) — il n'y a rien à differ à l'exécution car le compilateur sait déjà, statiquement, quel nœud DOM dépend de quelle variable. C'est ce qui explique à la fois l'absence de runtime de framework à charger et la rapidité des mises à jour.
- **SvelteKit : SSR, SSG et adapters** : comme Nuxt/Nitro, SvelteKit compile vers différentes cibles de déploiement via des *adapters* (`adapter-node` pour un serveur classique, `adapter-static` pour du SSG pur, `adapter-vercel`/`adapter-cloudflare` pour de l'edge/serverless) — même logique de "un seul code, plusieurs cibles" que Nitro (voir [`../nuxtjs/`](../nuxtjs/)).
- **Form actions et progressive enhancement** : `+page.server.js` peut exporter des `actions` qui traitent une soumission de formulaire **côté serveur**, fonctionnant nativement même si le JavaScript client n'est pas encore chargé (le formulaire POST classiquement), puis s'améliorant automatiquement (sans rechargement de page complet) une fois le JS actif — une philosophie de résilience différente du tout-JS par défaut de React/Vue.

```js
// +page.server.js
export const actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    // traiter data.get('email')...
    return { success: true };
  }
};
```

- **Transitions et animations natives** : `transition:`, `in:`/`out:`, et `animate:` sont des directives intégrées au langage pour animer l'apparition/disparition/réordonnancement d'éléments — sans bibliothèque tierce (Framer Motion en React, par exemple), directement compilées en code d'animation optimisé.

```svelte
<script>
  import { fade } from 'svelte/transition';
  let visible = $state(true);
</script>

{#if visible}
  <div transition:fade={{ duration: 200 }}>Contenu qui apparaît/disparaît en fondu</div>
{/if}
```

- **Tests de composants** : `@testing-library/svelte` suit la même philosophie que Testing Library sur React/Vue (tester le comportement perçu par l'utilisateur, pas les détails d'implémentation), combiné à Vitest.

```js
import { render, screen, fireEvent } from '@testing-library/svelte';
import Counter from './Counter.svelte';

test('incrémente au clic', async () => {
  render(Counter);
  await fireEvent.click(screen.getByRole('button'));
  expect(screen.getByText(/Compteur : 1/)).toBeInTheDocument();
});
```

Voir [`../testing/`](../testing/) pour la stratégie de test générale.

## 6. Commandes / syntaxe à connaître

```bash
npx sv create mon-projet     # scaffolding d'un nouveau projet SvelteKit
npm install                   # installer les dépendances
npm run dev                    # serveur de dev (Vite)
npm run build                   # build de production
npm run preview                  # prévisualiser le build de prod en local
npx vitest                         # lancer les tests
npx svelte-check                     # vérification TypeScript des .svelte
```

```svelte
<!-- Runes essentielles -->
let x = $state(valeur)
let y = $derived(expression)
$effect(() => { /* effet */ return () => { /* cleanup */ } })
let { prop1, prop2 = defaut } = $props()

<!-- Contrôle de flux dans le markup -->
{#if cond}...{:else if cond2}...{:else}...{/if}
{#each liste as item (item.id)}...{/each}
```

## 7. Exercices

Voir [`exercices/niveau-1.md`](exercices/niveau-1.md), [`niveau-2.md`](exercices/niveau-2.md), [`niveau-3.md`](exercices/niveau-3.md). Corrections séparées dans [`solutions/`](solutions/).

## 8. Mini-projet

**Gestionnaire de tâches (todo-list) avec filtres et persistance locale**

Reprendre le mini-projet Vue ([`../vuejs/README.md`](../vuejs/README.md)) en Svelte, pour comparer directement les deux approches sur le même exercice :
- Composant `TodoApp` avec `$state` pour la liste de tâches.
- Ajouter, cocher, supprimer une tâche ; filtrer (toutes/actives/terminées) via un `$derived`.
- Persister la liste dans `localStorage` via un `$effect` qui se déclenche à chaque changement de la liste.
- Bonus : découper en sous-composants (`TodoItem`, `TodoFilters`) avec props/callbacks, et migrer vers des routes SvelteKit (`/`, `/stats`) avec une fonction `load` pour précharger des statistiques si la todo-list doit être partagée entre plusieurs pages.

Objectif : ressentir directement, sur la même application, la différence de verbosité et de modèle mental entre les runes Svelte et la Composition API Vue.

## Checklist

- [ ] Comprendre les fondamentaux (compilation sans Virtual DOM, runes `$state`/`$derived`/`$effect`)
- [ ] Savoir créer un projet SvelteKit
- [ ] Maîtriser la syntaxe principale (`{#if}`, `{#each}`, événements, `$props()`, `bind:value`)
- [ ] Comprendre les concepts importants (stores, routing par fichiers, fonctions `load`)
- [ ] Savoir debugger (Svelte DevTools, messages du compilateur)
- [ ] Connaître les bonnes pratiques (clé stable sur `{#each}`, `$derived` plutôt qu'`$effect` pour du calcul pur)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (modèle de compilation, adapters SvelteKit, form actions, transitions natives)

## 10. Ressources

- [Documentation officielle Svelte](https://svelte.dev/docs/svelte) — référence complète du langage et des runes.
- [Documentation officielle SvelteKit](https://svelte.dev/docs/kit) — référence du méta-framework (routing, `load`, adapters).
- [Svelte Tutorial (interactif)](https://svelte.dev/tutorial) — exercices guidés officiels dans le navigateur.
- [Testing Library — Svelte](https://testing-library.com/docs/svelte-testing-library/intro/) — philosophie et API de test.
- [`../vuejs/`](../vuejs/) et [`../nuxtjs/`](../nuxtjs/) pour la comparaison directe avec l'approche framework + méta-framework séparés.
