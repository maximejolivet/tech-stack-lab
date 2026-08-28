# Vue.js — Exercices niveau 2 (Intermédiaire)

## Exercice 1 — Liste filtrable avec `computed`

À partir d'une liste de produits (`{ id, name, price, category }`) stockée dans un `ref`, crée :
- un `ref` `selectedCategory` (`'all'` par défaut) piloté par un `<select>` ;
- un `computed` `filteredProducts` qui retourne les produits filtrés par catégorie ;
- un `computed` `totalPrice` qui calcule la somme des prix affichés.

## Exercice 2 — Composable `useFetch`

Crée un composable `composables/useFetch.js` qui prend une URL en paramètre et retourne `{ data, error, loading }` (des `ref`), en effectuant l'appel `fetch` dans un `onMounted` (ou directement dans le composable). Utilise-le dans un composant pour afficher une liste récupérée depuis une API publique (ex. `https://jsonplaceholder.typicode.com/users`).

## Exercice 3 — Formulaire avec validation

Crée un formulaire d'inscription (email, mot de passe, confirmation) avec :
- un `ref` par champ ;
- un `computed` `isValid` qui vérifie que l'email contient `@`, que le mot de passe fait au moins 8 caractères, et que les deux mots de passe correspondent ;
- le bouton de soumission désactivé (`:disabled`) tant que `isValid` est `false` ;
- affichage des erreurs correspondantes sous chaque champ.

## Exercice 4 — Composants avec slots

Crée un composant `Modal.vue` réutilisable avec un slot nommé `header` et un slot par défaut pour le contenu, plus un bouton de fermeture géré par emit. Utilise-le pour afficher au moins deux modales différentes avec des contenus différents depuis le même composant parent.
