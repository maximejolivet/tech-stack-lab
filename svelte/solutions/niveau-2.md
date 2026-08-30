# Svelte — Solutions niveau 2

## Exercice 1 — Liste filtrable avec `$derived`

```svelte
<script>
  let products = $state([
    { id: 1, name: 'Clavier', price: 50, category: 'peripherique' },
    { id: 2, name: 'Écran', price: 200, category: 'ecran' },
    { id: 3, name: 'Souris', price: 25, category: 'peripherique' },
  ]);
  let selectedCategory = $state('all');

  let filteredProducts = $derived(
    selectedCategory === 'all'
      ? products
      : products.filter(p => p.category === selectedCategory)
  );
  let totalPrice = $derived(filteredProducts.reduce((sum, p) => sum + p.price, 0));
</script>

<select bind:value={selectedCategory}>
  <option value="all">Toutes</option>
  <option value="peripherique">Périphériques</option>
  <option value="ecran">Écrans</option>
</select>
<ul>
  {#each filteredProducts as p (p.id)}
    <li>{p.name} — {p.price}€</li>
  {/each}
</ul>
<p>Total : {totalPrice}€</p>
```

## Exercice 2 — Formulaire avec `bind:value` et validation

```svelte
<script>
  let email = $state('');
  let password = $state('');
  let confirmPassword = $state('');

  let emailValid = $derived(email.includes('@'));
  let passwordValid = $derived(password.length >= 8);
  let passwordsMatch = $derived(password === confirmPassword);
  let isValid = $derived(emailValid && passwordValid && passwordsMatch);
</script>

<form onsubmit={(e) => e.preventDefault()}>
  <input bind:value={email} placeholder="Email" />
  {#if email && !emailValid}<p>Email invalide</p>{/if}

  <input bind:value={password} type="password" placeholder="Mot de passe" />
  {#if password && !passwordValid}<p>8 caractères minimum</p>{/if}

  <input bind:value={confirmPassword} type="password" placeholder="Confirmation" />
  {#if confirmPassword && !passwordsMatch}<p>Les mots de passe ne correspondent pas</p>{/if}

  <button disabled={!isValid}>S'inscrire</button>
</form>
```

## Exercice 3 — Composition avec `{#snippet}`

```svelte
<!-- Modal.svelte -->
<script>
  let { header, children, onClose } = $props();
</script>

<div class="modal-overlay" onclick={onClose}>
  <div class="modal">
    <header>{@render header?.()}<button onclick={onClose}>×</button></header>
    <main>{@render children()}</main>
  </div>
</div>
```

```svelte
<!-- Parent -->
<script>
  import Modal from './Modal.svelte';
  let showConfirm = $state(false);
  let showInfo = $state(false);
</script>

<button onclick={() => showConfirm = true}>Supprimer</button>
{#if showConfirm}
  <Modal onClose={() => showConfirm = false}>
    {#snippet header()}Confirmation{/snippet}
    Voulez-vous vraiment supprimer cet élément ?
  </Modal>
{/if}

<button onclick={() => showInfo = true}>Infos</button>
{#if showInfo}
  <Modal onClose={() => showInfo = false}>
    {#snippet header()}À propos{/snippet}
    Cette application a été créée avec Svelte.
  </Modal>
{/if}
```

## Exercice 4 — Store partagé

```js
// stores/cart.js
import { writable } from 'svelte/store';

export const cart = writable([]);
export function addItem(item) {
  cart.update(items => [...items, item]);
}
export function removeItem(id) {
  cart.update(items => items.filter(i => i.id !== id));
}
```

```svelte
<!-- CartIcon.svelte -->
<script>
  import { cart } from '../stores/cart.js';
</script>
<span>🛒 {$cart.length}</span>
```

```svelte
<!-- CartView.svelte -->
<script>
  import { cart, addItem } from '../stores/cart.js';
</script>
<button onclick={() => addItem({ id: Date.now(), name: 'Article' })}>Ajouter</button>
<p>{$cart.length} articles dans le panier</p>
```

## Exercice 5 — Routing SvelteKit avec layout partagé

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  let { children } = $props();
</script>

<header>
  <a href="/">Accueil</a>
  <a href="/about">À propos</a>
</header>
<main>{@render children()}</main>
```

```svelte
<!-- src/routes/+page.svelte -->
<h1>Accueil</h1>
```

```svelte
<!-- src/routes/about/+page.svelte -->
<h1>À propos</h1>
```

**Points clés** : `$derived` remplace `computed`, sans fonction à appeler explicitement ; les `{#snippet}` remplacent les slots nommés de Vue, `children` remplace le slot par défaut ; un store `writable` reste réactif partout où il est importé, souscription automatique via le préfixe `$` ; `+layout.svelte` avec `children` enveloppe toutes les routes filles, comme un layout Nuxt.
