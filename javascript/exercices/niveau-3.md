# Exercices JavaScript — Niveau 3 : Avancé

Objectif : problèmes proches de situations professionnelles, combinant event loop, performance, patterns et architecture.

## 1. Débogage d'ordre d'exécution

Prédire l'ordre exact d'affichage, puis expliquer pourquoi (microtasks vs macrotasks) :

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
Promise.resolve().then(() => console.log("5")).then(() => console.log("6"));
setTimeout(() => console.log("7"), 0);
```

## 2. Debounce

Implémenter une fonction générique `debounce(fn, delay)` (sans librairie) qui retourne une version "debouncée" de `fn`. L'utiliser sur un champ de recherche pour ne déclencher un appel API que 300ms après la dernière frappe. Gérer le cas où l'utilisateur tape puis quitte la page avant le délai (pas d'appel fantôme).

## 3. Cache avec mémoïsation

Écrire une fonction `memoize(fn)` générique qui met en cache le résultat d'un appel selon ses arguments (clé = `JSON.stringify(args)`), pour éviter de recalculer un résultat coûteux (ex. un calcul récursif ou un appel réseau) déjà obtenu avec les mêmes paramètres. Gérer le cas d'une fonction asynchrone (ne pas mettre en cache une Promise rejetée).

## 4. File d'attente de requêtes limitée en concurrence

Écrire une fonction `runWithConcurrencyLimit(tasks, limit)` qui exécute un tableau de fonctions asynchrones (`tasks: () => Promise`) en garantissant qu'au maximum `limit` tâches s'exécutent en parallèle à un instant donné, et retourne un tableau des résultats dans l'ordre d'origine.

## 5. Petit state manager observable

Implémenter un mini state manager `createStore(initialState)` qui expose `getState()`, `setState(partial)`, et `subscribe(listener)` (pattern Observer/PubSub). `setState` doit fusionner (pas remplacer) l'état, notifier tous les abonnés, et `subscribe` doit retourner une fonction de désabonnement. Réfléchir à l'immutabilité de l'état retourné par `getState()`.

## 6. Fuite mémoire

Ce code crée une fuite mémoire dans une single-page app où `renderWidget()` est appelé plusieurs fois sans jamais recharger la page. Identifier la fuite et la corriger :

```js
function renderWidget(container) {
  const data = new Array(1_000_000).fill("x"); // grosse donnée
  const button = document.createElement("button");
  button.textContent = "Cliquer";
  button.addEventListener("click", () => console.log(data.length));
  container.innerHTML = "";
  container.append(button);
}
```
