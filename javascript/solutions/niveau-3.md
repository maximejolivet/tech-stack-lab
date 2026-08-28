# Solutions — Niveau 3 : Avancé

## 1. Ordre d'exécution

Ordre affiché : `1, 4, 3, 5, 6, 2, 7`

Explication :
1. Le code synchrone s'exécute d'abord en entier : `1`, `4` (les `console.log` directs). Les `setTimeout` et `.then()` sont juste **enregistrés**, pas exécutés immédiatement.
2. Une fois le call stack vide, la **microtask queue** (Promises) est vidée entièrement avant toute macrotask : `3`, puis `5` (le `.then(() => console.log(6))` chaîné est ré-enfilé en microtask après l'exécution de `5`), donc `6`.
3. Seulement ensuite, les **macrotasks** (`setTimeout`) s'exécutent dans leur ordre d'enregistrement : `2`, puis `7`.

## 2. Debounce

```js
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId); // annule l'appel précédent en attente
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

const debouncedSearch = debounce((query) => searchApi(query), 300);
input.addEventListener("input", (e) => debouncedSearch(e.target.value));
```

`clearTimeout` sur chaque frappe garantit qu'un seul appel réel a lieu, 300ms après la dernière frappe. Si l'utilisateur quitte la page, le timer en attente est simplement abandonné avec la page (pas d'appel fantôme puisque rien ne s'exécute plus).

## 3. Mémoïsation

```js
function memoize(fn) {
  const cache = new Map();
  return async function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);

    const result = fn.apply(this, args);
    // si fn est async, result est une Promise : on ne met en cache
    // qu'une fois résolue avec succès (pas un rejet)
    try {
      const value = await result;
      cache.set(key, value);
      return value;
    } catch (err) {
      cache.delete(key); // ne pas garder un échec en cache
      throw err;
    }
  };
}
```

## 4. File d'attente à concurrence limitée

```js
async function runWithConcurrencyLimit(tasks, limit) {
  const results = new Array(tasks.length);
  let nextIndex = 0;

  async function worker() {
    while (nextIndex < tasks.length) {
      const currentIndex = nextIndex++;
      results[currentIndex] = await tasks[currentIndex]();
    }
  }

  const workers = Array.from({ length: limit }, () => worker());
  await Promise.all(workers);
  return results;
}
```

`limit` workers tournent en parallèle, chacun prend la prochaine tâche disponible dès qu'il est libre — garantit au plus `limit` promesses en vol simultanément, tout en préservant l'ordre des résultats via `results[currentIndex]`.

## 5. Store observable

```js
function createStore(initialState) {
  let state = { ...initialState };
  const listeners = new Set();

  return {
    getState: () => ({ ...state }), // copie : évite qu'un consommateur mute l'état interne
    setState(partial) {
      state = { ...state, ...partial };
      listeners.forEach((listener) => listener(state));
    },
    subscribe(listener) {
      listeners.add(listener);
      return () => listeners.delete(listener); // désabonnement
    },
  };
}
```

## 6. Fuite mémoire

La fuite : chaque appel à `renderWidget()` crée un tableau `data` d'1 million d'éléments capturé par la closure du listener du bouton. Le `container.innerHTML = ""` retire l'ancien bouton du DOM, **mais** si une référence au listener (ou au bouton) traîne encore ailleurs (ou selon le navigateur/GC), le tableau `data` associé peut rester en mémoire plus longtemps que nécessaire — et à chaque re-render, un nouveau tableau d'1M d'éléments est alloué sans que l'ancien ne soit garanti d'être libéré rapidement.

Correction — ne pas capturer de grosse donnée inutilement dans la closure du listener, et retirer explicitement l'ancien listener avant de vider le conteneur :

```js
let currentButton = null;

function renderWidget(container) {
  const dataLength = new Array(1_000_000).fill("x").length; // on ne garde que ce dont on a besoin

  if (currentButton) {
    currentButton.removeEventListener("click", currentButton._handler);
  }

  const button = document.createElement("button");
  button.textContent = "Cliquer";
  const handler = () => console.log(dataLength);
  button.addEventListener("click", handler);
  button._handler = handler;

  container.innerHTML = "";
  container.append(button);
  currentButton = button;
}
```

Leçon générale : ne capturer dans une closure/un listener que les données réellement nécessaires à l'exécution future, pas l'objet entier "au cas où".
