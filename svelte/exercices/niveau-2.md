# Svelte — Exercices niveau 2 (Intermédiaire)

## Exercice 1 — Liste filtrable avec `$derived`

À partir d'une liste de produits (`{ id, name, price, category }`) stockée dans une variable `$state`, crée :
- une variable `$state` `selectedCategory` (`'all'` par défaut) pilotée par un `<select bind:value>` ;
- une valeur `$derived` `filteredProducts` qui retourne les produits filtrés par catégorie ;
- une valeur `$derived` `totalPrice` qui calcule la somme des prix affichés.

## Exercice 2 — Formulaire avec `bind:value` et validation

Crée un formulaire d'inscription (email, mot de passe, confirmation) avec :
- une variable `$state` par champ, liée via `bind:value` ;
- une valeur `$derived` `isValid` qui vérifie que l'email contient `@`, que le mot de passe fait au moins 8 caractères, et que les deux mots de passe correspondent ;
- le bouton de soumission désactivé (`disabled={!isValid}`) tant que `isValid` est faux ;
- affichage des erreurs correspondantes sous chaque champ.

## Exercice 3 — Composition avec `{#snippet}`

Crée un composant `Modal.svelte` réutilisable avec un `{#snippet header()}` et des `children` pour le contenu principal, plus un bouton de fermeture appelant une prop-fonction `onClose`. Utilise-le pour afficher au moins deux modales différentes avec des contenus différents depuis le même composant parent.

## Exercice 4 — Store partagé

Crée un store `stores/cart.js` avec `writable([])` et des fonctions `addItem`/`removeItem`. Utilise ce store depuis deux composants sans lien parent-enfant (ex. un `CartIcon` et une `CartView`), et vérifie que l'état reste synchronisé entre les deux (préfixe `$` dans le markup pour la souscription automatique).

## Exercice 5 — Routing SvelteKit avec layout partagé

Dans un projet SvelteKit, crée deux routes `src/routes/+page.svelte` (accueil) et `src/routes/about/+page.svelte`, ainsi qu'un `src/routes/+layout.svelte` avec un header contenant des liens de navigation entre les deux pages, affiché sur les deux routes.
