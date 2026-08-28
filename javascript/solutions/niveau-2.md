# Solutions — Niveau 2 : Intermédiaire

## 1. Closures

```js
function createCounter(start = 0) {
  let value = start; // privée : inaccessible directement en dehors de la closure
  return {
    increment: () => ++value,
    decrement: () => --value,
    getValue: () => value,
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getValue()); // 2
console.log(counter.value);       // undefined — bien privée
```

## 2. `this`

Problème : `setInterval` appelle le callback comme une fonction "simple" (pas une méthode), donc `this` n'est pas `timer` mais `undefined` (mode strict) ou l'objet global.

**Approche 1 — `bind`** :

```js
start() {
  setInterval(function () {
    this.seconds++;
    console.log(this.seconds);
  }.bind(this), 1000); // lie `this` explicitement au `this` de start() (= timer)
}
```

**Approche 2 — arrow function** (recommandée, plus lisible) :

```js
start() {
  setInterval(() => {
    this.seconds++; // arrow function : `this` capturé lexicalement = timer
    console.log(this.seconds);
  }, 1000);
}
```

## 3. Promises

```js
function fetchWithTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Timeout")), ms)
  );
  return Promise.race([promise, timeout]);
}
```

## 4. Async/Await

```js
async function loadDashboard(id) {
  const [user, posts, comments] = await Promise.all([
    fetchUser(id),
    fetchPosts(id),
    fetchComments(id),
  ]);
  return { user, posts, comments };
}
```

## 5. Gestion des erreurs

```js
class HttpError extends Error {
  constructor(message, status) {
    super(message);
    this.name = "HttpError";
    this.status = status;
  }
}

async function safeFetch(url) {
  const response = await fetch(url);
  if (!response.ok) {
    throw new HttpError(`Requête échouée : ${response.status}`, response.status);
  }
  return response.json();
}
```

## 6. Modules

```js
// cart.js
let items = [];

export function addItem(name, price) {
  items.push({ name, price });
}

export function getTotal() {
  return items.reduce((sum, i) => sum + i.price, 0);
}
```

```js
// main.js
import { addItem, getTotal } from "./cart.js";

addItem("Livre", 15);
addItem("Stylo", 2);
console.log(getTotal());
```

Pas d'export par défaut nécessaire ici : deux fonctions de même importance, un export nommé pour chacune est plus explicite à l'import.

## 7. Events (DOM)

```js
document.getElementById("list").addEventListener("click", (event) => {
  if (event.target.matches(".item")) {
    console.log("Clic sur :", event.target.textContent);
  }
});
```

Un seul listener sur le parent `#list` : fonctionne pour les `<li>` déjà présents ET ceux ajoutés dynamiquement plus tard, sans avoir à ré-attacher d'écouteur.
