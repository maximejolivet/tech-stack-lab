# Exercices JavaScript — Niveau 2 : Intermédiaire

Objectif : combiner plusieurs notions (closures, `this`, promises, async/await, modules, gestion d'erreurs).

## 1. Closures

Écrire une fonction `createCounter(start = 0)` qui retourne un objet `{ increment, decrement, getValue }` où `value` est une variable privée inaccessible directement de l'extérieur (encapsulation par closure).

## 2. `this`

Expliquer pourquoi ce code ne fonctionne pas comme attendu, et corriger avec **deux** approches différentes (bind, et arrow function) :

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(function () {
      this.seconds++; // bug : `this` n'est pas `timer` ici
      console.log(this.seconds);
    }, 1000);
  },
};
timer.start();
```

## 3. Promises

Écrire une fonction `fetchWithTimeout(promise, ms)` qui rejette avec une erreur `"Timeout"` si `promise` ne se résout pas dans le délai `ms`, en utilisant `Promise.race`.

## 4. Async/Await

Étant donné trois fonctions asynchrones indépendantes `fetchUser(id)`, `fetchPosts(id)`, `fetchComments(id)` (simulées avec des `setTimeout`), écrire une fonction `loadDashboard(id)` qui les exécute **en parallèle** (pas en série) et retourne `{ user, posts, comments }`.

## 5. Gestion des erreurs

Créer une classe `HttpError extends Error` avec une propriété `status`. Écrire une fonction `safeFetch(url)` qui : lève une `HttpError` si `response.ok` est `false`, et retourne les données JSON sinon.

## 6. Modules

Découper ce fichier unique en deux modules ESM : `cart.js` (logique métier : ajouter un article, calculer le total) et `main.js` (utilisation). Utiliser un export nommé et un export par défaut à bon escient.

```js
let items = [];
function addItem(name, price) { items.push({ name, price }); }
function getTotal() { return items.reduce((sum, i) => sum + i.price, 0); }
addItem("Livre", 15);
addItem("Stylo", 2);
console.log(getTotal());
```

## 7. Events (DOM)

Sur une liste `<ul id="list">` avec des `<li class="item">` ajoutés dynamiquement, écrire le JS qui gère un clic sur n'importe quel `<li>` (présent ou futur) avec **un seul** `addEventListener`, en utilisant la délégation d'événements.
