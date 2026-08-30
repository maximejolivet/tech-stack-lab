# Svelte — Exercices niveau 3 (Avancé)

## Exercice 1 — Chargement de données avec `load`

Dans une route SvelteKit `src/routes/users/[id]/+page.js`, écris une fonction `load({ fetch, params })` qui récupère un utilisateur depuis `https://jsonplaceholder.typicode.com/users/{id}`. Affiche le résultat dans `+page.svelte` via la prop `data` injectée automatiquement.

## Exercice 2 — Form action avec progressive enhancement

Crée une route `src/routes/contact/+page.server.js` exportant une action `default` qui reçoit un formulaire (`request.formData()`), valide qu'un champ `email` est présent, et retourne `{ success: true }` ou une erreur. Construis le formulaire correspondant dans `+page.svelte` (fonctionnel même sans JavaScript, méthode POST classique).

## Exercice 3 — Transition sur un élément conditionnel

Crée un composant avec un bouton qui bascule l'affichage d'un panneau (`{#if}`), en enveloppant le panneau d'une directive `transition:fade` (import depuis `svelte/transition`) avec une durée de 300ms. Ajoute une deuxième version utilisant `in:fly`/`out:fade` pour des transitions d'entrée/sortie différentes.

## Exercice 4 — Comparaison du modèle de compilation

Sans écrire de code, explique en 5-6 lignes pourquoi un composant Svelte équivalent à un composant React/Vue produit généralement moins de JavaScript envoyé au navigateur. Appuie-toi sur la différence entre "générer des instructions de mise à jour DOM au build" (Svelte) et "differ un arbre virtuel à l'exécution" (React/Vue, voir [`../react/`](../react/) section reconciliation, ou [`../vuejs/`](../vuejs/)). Donne un exemple concret d'application où ce gain de bundle a un impact réel (ex. site à fort trafic mobile, connexions lentes).

## Exercice 5 — Store dérivé

À partir du store `cart` de l'exercice 4 du niveau 2, crée un store dérivé (`derived` de `svelte/store`) `cartTotal` qui calcule automatiquement le total du panier à chaque changement de `cart`, sans dupliquer la logique de calcul dans chaque composant qui affiche le total.
