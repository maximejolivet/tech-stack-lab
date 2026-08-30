# Svelte — Solutions niveau 3

## Exercice 1 — Chargement de données avec `load`

```js
// src/routes/users/[id]/+page.js
export async function load({ fetch, params }) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${params.id}`);
  const user = await res.json();
  return { user };
}
```

```svelte
<!-- src/routes/users/[id]/+page.svelte -->
<script>
  let { data } = $props();
</script>

<h1>{data.user.name}</h1>
<p>{data.user.email}</p>
```

## Exercice 2 — Form action avec progressive enhancement

```js
// src/routes/contact/+page.server.js
export const actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    const email = data.get('email');
    if (!email) {
      return { success: false, error: 'Email requis' };
    }
    return { success: true };
  }
};
```

```svelte
<!-- src/routes/contact/+page.svelte -->
<script>
  let { form } = $props();
</script>

<form method="POST">
  <input name="email" type="email" placeholder="Email" />
  <button type="submit">Envoyer</button>
</form>

{#if form?.success}
  <p>Message envoyé !</p>
{:else if form?.error}
  <p>{form.error}</p>
{/if}
```

Ce formulaire fonctionne dès la première charge, même sans JavaScript actif (POST classique traité par `+page.server.js`) : c'est la progressive enhancement, activable ensuite pour éviter le rechargement complet via `use:enhance` de `$app/forms`.

## Exercice 3 — Transition sur un élément conditionnel

```svelte
<script>
  import { fade, fly } from 'svelte/transition';
  let visible = $state(false);
</script>

<button onclick={() => visible = !visible}>Basculer</button>

{#if visible}
  <div transition:fade={{ duration: 300 }}>Panneau avec fondu</div>
{/if}

{#if visible}
  <div in:fly={{ y: -20, duration: 300 }} out:fade={{ duration: 150 }}>
    Panneau avec entrée en glissement, sortie en fondu
  </div>
{/if}
```

## Exercice 4 — Comparaison du modèle de compilation

React et Vue construisent, à chaque changement d'état, un arbre virtuel (Virtual DOM) représentant l'UI souhaitée, puis le comparent (diff) à l'arbre précédent pour déterminer les mutations DOM réelles à appliquer — ce diffing s'exécute **dans le navigateur**, à l'exécution, et nécessite d'embarquer tout le runtime capable de le faire (le "framework" proprement dit). Svelte, en tant que compilateur, analyse le composant **au build** et sait déjà statiquement que, par exemple, changer `count` ne touche qu'un seul nœud texte précis du DOM — il génère directement le code JS qui fait cette mutation ciblée, sans jamais construire ni comparer d'arbre à l'exécution. Le gain de bundle vient du fait qu'aucun moteur de diffing générique n'a besoin d'être téléchargé : seul le code spécifique au composant, déjà optimisé, est envoyé. Sur un site à fort trafic mobile avec des connexions lentes (marché émergent, zone rurale, réseau 3G), quelques dizaines de kilo-octets de runtime en moins par page se traduisent directement en temps de chargement et en taux de rebond réduits, un enjeu bien documenté dans [`../accessibility-performance/`](../accessibility-performance/).

## Exercice 5 — Store dérivé

```js
// stores/cart.js
import { writable, derived } from 'svelte/store';

export const cart = writable([]);

export const cartTotal = derived(cart, ($cart) =>
  $cart.reduce((sum, item) => sum + item.price, 0)
);
```

```svelte
<script>
  import { cartTotal } from '../stores/cart.js';
</script>
<p>Total : {$cartTotal}€</p>
```

`derived` recalcule automatiquement `cartTotal` à chaque mise à jour de `cart`, sans dupliquer le calcul dans chaque composant consommateur — même principe qu'un getter Pinia calculé à partir du state.
