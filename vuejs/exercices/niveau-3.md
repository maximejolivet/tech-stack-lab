# Vue.js — Exercices niveau 3 (Avancé)

## Exercice 1 — Store Pinia partagé

Crée un store Pinia `useCartStore` (ajout/suppression d'articles, quantité, total calculé via un getter). Utilise ce store depuis au moins deux composants sans lien parent-enfant (ex. un `CartIcon` dans un header et une page `CartView`), et vérifie que l'état reste synchronisé entre les deux.

## Exercice 2 — Composant de liste virtualisée avec `v-memo`

À partir d'une liste de 1000 éléments générée aléatoirement, affiche-la avec `v-for` et utilise `v-memo` pour éviter les re-rendus inutiles des items dont les données n'ont pas changé. Mesure (via Vue Devtools ou `console.time`) l'impact avant/après sur le nombre de nœuds DOM re-rendus lors d'une mise à jour partielle de la liste.

## Exercice 3 — Chargement asynchrone avec `Suspense`

Crée un composant `UserProfile.vue` dont le `<script setup>` utilise un top-level `await` pour charger les données d'un utilisateur avant le rendu. Enveloppe son utilisation dans un `<Suspense>` avec un fallback de chargement, et gère le cas d'erreur avec un `onErrorCaptured` dans le composant parent.

## Exercice 4 — Mini state machine pour un formulaire multi-étapes

Implémente un formulaire en 3 étapes (infos personnelles → adresse → confirmation) où :
- l'état de l'étape courante et les données de chaque étape sont gérés dans un composable `useMultiStepForm()` ;
- on ne peut pas passer à l'étape suivante si l'étape courante n'est pas valide ;
- on peut revenir en arrière sans perdre les données déjà saisies ;
- bonus : persiste l'état dans `sessionStorage` pour survivre à un rafraîchissement de page.
