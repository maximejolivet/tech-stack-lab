# JavaScript

## 1. Introduction

JavaScript est le langage de programmation du web : c'est le seul langage exécuté nativement par tous les navigateurs, et il tourne aussi côté serveur via Node.js. Ce dossier couvre le langage **vanilla** (syntaxe, sémantique, APIs navigateur de base : DOM, Events, Fetch) — le typage statique est traité séparément dans [`../typescript/`](../typescript/), et les frameworks (React, Vue...) ont chacun leur propre dossier.

**À quoi sert-il ?**
- Rendre une page interactive côté client (manipulation du DOM, gestion d'événements).
- Faire des appels réseau asynchrones (Fetch, WebSockets).
- Écrire du backend (Node.js), des scripts CLI, ou même des apps mobiles/desktop (React Native, Electron).

**Où se situe-t-il dans une architecture web ?**
Historiquement uniquement côté client (navigateur), JS est aujourd'hui full-stack : front (DOM/frameworks), back (Node.js/API), tooling (bundlers, CLI), et même infra (scripts d'automatisation).

**Avantages**
- Langage unique pour front et back (JS/Node).
- Écosystème npm énorme, exécution partout (navigateur, serveur, edge).
- Asynchrone par nature (event loop), adapté à l'I/O intensif.

**Limites**
- Typage dynamique et faible → bugs runtime (d'où TypeScript en pratique professionnelle).
- Nombreux pièges historiques (coercition, `this`, `==` vs `===`) hérités de choix de design des années 90.
- Mono-thread : le calcul lourd bloque l'event loop (parade : Web Workers, découpage des tâches).

## 2. Prérequis

- Aucune connaissance préalable de JS n'est requise, mais une expérience générale de programmation (variables, fonctions, boucles dans un autre langage comme PHP) aide à aller plus vite sur les bases.
- Un navigateur avec DevTools (Chrome/Firefox) pour exécuter les exemples en console.
- Node.js installé (pour exécuter du JS hors navigateur, voir [`../nodejs/`](../nodejs/)) — optionnel pour ce dossier mais pratique pour tester rapidement.

## 3. Rappel des bases 🟢

### 01 - Variables

**Explication** — Trois mots-clés : `var` (obsolète, à éviter), `let` (réassignable, scope de bloc), `const` (non réassignable, scope de bloc).

```js
const name = "Max";     // ne peut pas être réassigné
let count = 0;           // peut être réassigné
count = 1;                // OK
```

**Cas d'usage** : `const` par défaut systématiquement, `let` seulement si la variable doit changer de valeur (compteur, accumulateur).

**Erreur fréquente** : croire que `const` rend un objet immuable. `const` empêche seulement la *réassignation* de la variable, pas la mutation de son contenu :

```js
const user = { name: "Max" };
user.name = "Alex"; // OK, autorisé
user = {};           // Erreur : Assignment to constant variable
```

**Bonne pratique** : bannir `var` (hoisting et scope fonction source de bugs), préférer `const`, n'utiliser `let` qu'en cas de réassignation nécessaire.

### 02 - Types

**Explication** — JS a 7 types primitifs (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) et un type composite (`object`, qui inclut les arrays et fonctions). Le typage est **dynamique** (la variable n'a pas de type fixe) et **faible** (conversions implicites entre types).

```js
typeof "hello";   // "string"
typeof 42;        // "number"
typeof true;      // "boolean"
typeof undefined; // "undefined"
typeof null;      // "object" (bug historique du langage, à connaître)
typeof {};         // "object"
typeof [];         // "object" (utiliser Array.isArray() pour distinguer)
```

**`null` vs `undefined`** : `undefined` = valeur jamais assignée ; `null` = absence de valeur assignée intentionnellement.

**Erreur fréquente** : utiliser `==` (égalité faible, avec coercition de type) au lieu de `===` (égalité stricte).

```js
"5" == 5;   // true  (coercition)
"5" === 5;  // false (types différents)
null == undefined;  // true
null === undefined; // false
```

**Bonne pratique** : toujours utiliser `===` / `!==`, sauf cas très spécifique documenté. Utiliser `Number()`, `String()`, `Boolean()` pour des conversions explicites plutôt que de compter sur la coercition implicite.

### 03 - Conditions

**Explication** — `if/else`, `switch`, et opérateurs de logique courte (`&&`, `||`, `??`).

```js
if (age >= 18) {
  console.log("majeur");
} else if (age >= 13) {
  console.log("ado");
} else {
  console.log("enfant");
}

switch (role) {
  case "admin":
    grantFullAccess();
    break;
  case "editor":
    grantEditAccess();
    break;
  default:
    grantReadAccess();
}
```

**Valeurs falsy** (évaluées à `false` dans un contexte booléen) : `false`, `0`, `""`, `null`, `undefined`, `NaN`. Tout le reste est truthy, y compris `"0"`, `[]`, `{}`.

**Opérateur `??` (nullish coalescing)** vs `||` : `??` ne remplace que `null`/`undefined`, contrairement à `||` qui remplace toute valeur falsy.

```js
const count = 0;
count || 10;  // 10  (0 est falsy → piège si 0 est une valeur légitime)
count ?? 10;  // 0   (0 n'est ni null ni undefined → conservé)
```

**Erreur fréquente** : oublier le `break` dans un `switch` (fall-through non désiré).

**Bonne pratique** : utiliser `??` pour les valeurs par défaut sur des données potentiellement à `0`/`""` légitimes ; préférer une structure `Map`/objet à un `switch` long pour du dispatch de logique.

### 04 - Boucles

**Explication** — `for` classique, `for...of` (itère sur les valeurs d'un itérable), `for...in` (itère sur les clés énumérables d'un objet), `while`, `do...while`.

```js
const fruits = ["pomme", "poire", "kiwi"];

for (const fruit of fruits) console.log(fruit);        // valeurs
for (const index in fruits) console.log(index);        // clés/index (string !)

fruits.forEach((fruit, i) => console.log(i, fruit));    // méthode de tableau
```

**Erreur fréquente** : utiliser `for...in` sur un array (itère aussi sur des propriétés héritées et donne des index en `string`, pas en `number`). Réserver `for...in` aux objets, `for...of` aux arrays/itérables.

**Bonne pratique** : préférer les méthodes fonctionnelles (`map`, `filter`, `reduce`, `forEach`) à une boucle manuelle quand elles expriment mieux l'intention ; utiliser `for...of` pour une boucle simple avec `break`/`continue` (que `forEach` ne permet pas nativement).

### 05 - Fonctions

**Explication** — Trois syntaxes : déclaration de fonction (hoistée), expression de fonction, arrow function (`this` lexical, pas de hoisting).

```js
function add(a, b) { return a + b; }          // déclaration, hoistée
const sub = function (a, b) { return a - b; }; // expression, pas hoistée
const mul = (a, b) => a * b;                   // arrow function

function greet(name = "invité") {              // paramètre par défaut
  return `Bonjour ${name}`;
}

function sum(...numbers) {                     // rest parameters
  return numbers.reduce((total, n) => total + n, 0);
}
```

**Différence clé arrow vs classique** : une arrow function ne possède pas son propre `this` (elle capture celui du contexte englobant), n'a pas d'objet `arguments`, et ne peut pas être utilisée comme constructeur (`new`).

**Erreur fréquente** : définir une méthode d'objet en arrow function alors qu'elle a besoin de `this` :

```js
const counter = {
  value: 0,
  increment: () => { this.value++; }, // `this` n'est PAS `counter` ici, bug
};
```

**Bonne pratique** : méthode courte sans besoin de son propre `this` → arrow function ; méthode d'objet/classe → fonction classique (ou méthode raccourcie `increment() {}`).

### 06 - Scope

**Explication** — Trois portées : globale, fonction (`var`), bloc (`let`/`const`). Le scope détermine où une variable est accessible. JS utilise le **lexical scoping** : la portée est déterminée par l'emplacement du code dans le source, pas par l'ordre d'exécution.

```js
if (true) {
  var globalScope = "je fuis hors du bloc";
  let blockScope = "je reste dans le bloc";
}
console.log(globalScope); // OK (fuite hors du if)
console.log(blockScope);  // ReferenceError
```

**Closure** — une fonction "capture" les variables de son scope englobant, même après que celui-ci a fini de s'exécuter :

```js
function makeCounter() {
  let count = 0;
  return () => ++count; // capture `count` par référence
}
const counter = makeCounter();
counter(); // 1
counter(); // 2 — `count` a survécu à l'exécution de makeCounter()
```

**Erreur fréquente** : boucle avec `var` dans un callback asynchrone (`setTimeout`), toutes les closures partagent la même variable :

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // affiche 3, 3, 3 (pas 0, 1, 2)
}
// avec `let`, chaque itération a son propre scope de bloc → affiche 0, 1, 2
```

**Bonne pratique** : `let`/`const` systématiquement (scope de bloc prévisible) ; utiliser les closures consciemment pour l'encapsulation (état privé) plutôt que par accident.

### 07 - Arrays

**Explication** — Structure de données ordonnée et indexée. Méthodes essentielles à maîtriser : `map`, `filter`, `reduce`, `find`, `some`, `every`, `sort`, `includes`.

```js
const nums = [1, 2, 3, 4, 5];

nums.map(n => n * 2);              // [2, 4, 6, 8, 10] — transforme
nums.filter(n => n % 2 === 0);     // [2, 4]           — sélectionne
nums.reduce((acc, n) => acc + n, 0); // 15               — agrège
nums.find(n => n > 3);             // 4                — premier match
nums.some(n => n > 4);             // true             — au moins un
nums.every(n => n > 0);            // true             — tous
```

**Erreur fréquente** : `sort()` trie par défaut comme des **chaînes** (`[10, 2, 1].sort()` → `[1, 10, 2]`). Toujours fournir un comparateur pour des nombres :

```js
nums.sort((a, b) => a - b); // ordre numérique croissant
```

Autre piège : `map`/`filter`/`reduce` retournent un **nouveau tableau** (immutable), alors que `push`, `pop`, `splice`, `sort`, `reverse` **mutent** le tableau d'origine.

**Bonne pratique** : préférer les méthodes non-mutantes (`map`, `filter`, `[...arr, x]`, `toSorted()`) pour éviter les effets de bord, surtout dans un state géré par un framework front.

### 08 - Objects

**Explication** — Collection de paires clé-valeur. Syntaxe littérale, accès par point ou crochets (utile pour les clés dynamiques).

```js
const user = {
  name: "Max",
  age: 30,
  greet() { return `Salut, ${this.name}`; }, // méthode raccourcie
};

user.name;        // accès statique
user["age"];       // accès dynamique (utile si la clé est une variable)
const key = "age";
user[key];          // 30

Object.keys(user);    // ["name", "age", "greet"]
Object.values(user);   // ["Max", 30, function]
Object.entries(user);  // [["name", "Max"], ["age", 30], ...]
```

**Erreur fréquente** : comparer deux objets avec `===` en espérant une égalité "par valeur" — en JS, les objets sont comparés **par référence** :

```js
{ a: 1 } === { a: 1 }; // false, deux références différentes
```

**Bonne pratique** : utiliser `structuredClone()` (natif) pour une copie profonde, `{ ...obj }` pour une copie superficielle ; ne jamais muter un objet reçu en paramètre sans le documenter clairement.

### 09 - Destructuring

**Explication** — Extraire des valeurs d'un array ou des propriétés d'un objet en une seule expression.

```js
const { name, age } = user;                 // destructuring d'objet
const { name: userName } = user;            // renommage
const { role = "guest" } = user;            // valeur par défaut

const [first, second] = [1, 2, 3];          // destructuring d'array
const [, , third] = [1, 2, 3];              // on saute des éléments

function greet({ name, age }) {              // destructuring en paramètre
  return `${name}, ${age} ans`;
}
```

**Cas d'usage courant** : extraire des props dans un composant front, ou des champs précis d'une réponse API sans créer de variables intermédiaires.

**Erreur fréquente** : destructurer une valeur potentiellement `undefined`/`null` sans garde → `TypeError: Cannot destructure property`. Toujours s'assurer que l'objet source existe (valeur par défaut sur le paramètre, ou vérification préalable).

**Bonne pratique** : destructurer directement dans la signature de fonction pour des paramètres nommés explicites (meilleure lisibilité qu'une longue liste de paramètres positionnels).

### 10 - Spread / Rest

**Explication** — Le même opérateur `...`, avec deux usages opposés selon le contexte : **spread** (étale des éléments), **rest** (regroupe des éléments restants).

```js
// Spread — étale
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4];          // [1, 2, 3, 4]
const merged = { ...defaults, ...userOptions }; // fusion d'objets, droite prioritaire

// Rest — regroupe
function sum(...numbers) {              // numbers = tableau des arguments
  return numbers.reduce((a, b) => a + b, 0);
}
const { name, ...rest } = user;         // rest = tout sauf `name`
```

**Cas d'usage** : copie immuable d'un état avant modification (pattern très courant en React/Vue) ; fusion de configuration avec valeurs par défaut.

**Erreur fréquente** : le spread d'objet est une copie **superficielle** — un objet imbriqué reste partagé par référence :

```js
const original = { info: { age: 30 } };
const copy = { ...original };
copy.info.age = 31;
original.info.age; // 31 — muté aussi, car `info` est la même référence
```

**Bonne pratique** : pour un état imbriqué, spreader chaque niveau concerné, ou utiliser `structuredClone()`.

### 11 - Modules

**Explication** — ESM (`import`/`export`) est le standard moderne, remplaçant CommonJS (`require`/`module.exports`, encore dominant côté Node.js legacy).

```js
// math.js
export const PI = 3.14159;
export function square(x) { return x * x; }
export default function add(a, b) { return a + b; } // un seul export par défaut

// main.js
import add, { PI, square } from "./math.js";
import * as math from "./math.js"; // tout importer sous un namespace
```

**Cas d'usage** : découper le code en fichiers responsables d'une seule chose, permettre le tree-shaking (élimination du code mort au build).

**Erreur fréquente** : mélanger `import`/`export` (ESM) et `require`/`module.exports` (CommonJS) dans le même projet sans configuration adaptée → erreurs de résolution de module.

**Bonne pratique** : un export nommé par fonctionnalité (facilite le renommage à l'import et le tree-shaking), export par défaut réservé à l'export "principal" évident du fichier (ex. un composant).

### 12 - Promises

**Explication** — Objet représentant le résultat futur (réussi ou échoué) d'une opération asynchrone. Trois états : `pending`, `fulfilled`, `rejected`.

```js
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    if (!id) return reject(new Error("id manquant"));
    setTimeout(() => resolve({ id, name: "Max" }), 100);
  });
}

fetchUser(1)
  .then(user => console.log(user))
  .catch(err => console.error(err))
  .finally(() => console.log("terminé"));

// Combinateurs
Promise.all([fetchUser(1), fetchUser(2)]);      // échoue si UNE promesse échoue
Promise.allSettled([fetchUser(1), fetchUser(2)]); // attend toutes, sans échec global
Promise.race([fetchUser(1), timeout(500)]);      // la première qui se résout/rejette
```

**Erreur fréquente** : oublier de `return` une promesse dans une chaîne `.then()`, ce qui casse le chaînage (la suite ne s'exécute pas dans l'ordre attendu).

**Bonne pratique** : toujours terminer une chaîne de promesses par un `.catch()` (sinon un rejet devient une "unhandled promise rejection" silencieuse) ; utiliser `Promise.all` pour paralléliser des appels indépendants plutôt que les enchaîner en série.

### 13 - Async / Await

**Explication** — Sucre syntaxique au-dessus des Promises, pour écrire du code asynchrone avec une lecture séquentielle.

```js
async function loadUser(id) {
  try {
    const user = await fetchUser(id);
    return user;
  } catch (err) {
    console.error("Échec du chargement", err);
    throw err; // re-propager si l'appelant doit aussi réagir
  }
}
```

Une fonction `async` retourne **toujours** une Promise, même si elle `return` une valeur simple.

**Erreur fréquente** : utiliser `await` en série alors que les appels sont indépendants, ce qui sérialise inutilement des opérations parallélisables :

```js
// Lent — 2 appels en série (ex. 200ms + 200ms = 400ms)
const a = await fetchA();
const b = await fetchB();

// Rapide — 2 appels en parallèle (ex. max(200ms, 200ms) = 200ms)
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

**Bonne pratique** : `try/catch` autour de chaque `await` susceptible d'échouer avec une gestion spécifique ; `Promise.all` dès que plusieurs opérations async sont indépendantes.

### 14 - Gestion des erreurs

**Explication** — `try/catch/finally` pour le code synchrone et async/await ; objets `Error` (et sous-classes) pour transporter un message et une stack trace.

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

function validateAge(age) {
  if (age < 0) throw new ValidationError("Âge invalide", "age");
}

try {
  validateAge(-1);
} catch (err) {
  if (err instanceof ValidationError) {
    console.error(`Champ ${err.field} invalide : ${err.message}`);
  } else {
    throw err; // erreur inattendue, on ne l'avale pas silencieusement
  }
}
```

**Erreur fréquente** : attraper une erreur avec `catch` sans rien en faire (erreur "avalée") — masque des bugs et rend le debugging impossible.

**Bonne pratique** : créer des classes d'erreur métier (`ValidationError`, `NotFoundError`...) pour distinguer les cas dans le `catch` ; ne catcher que ce qu'on sait gérer, laisser remonter le reste.

### 15 - DOM

**Explication** — Le DOM (Document Object Model) est la représentation en mémoire du HTML sous forme d'arbre d'objets, manipulable en JS.

```js
const el = document.querySelector(".card");        // premier match
const all = document.querySelectorAll(".card");    // NodeList de tous les matchs

el.textContent = "Nouveau texte";  // texte brut (sûr, pas d'exécution HTML)
el.innerHTML = "<b>Texte</b>";      // parsé comme HTML (risque XSS si non fiable)

const div = document.createElement("div");
div.classList.add("active");
document.body.append(div);
```

**Erreur fréquente** : injecter du contenu utilisateur non échappé via `innerHTML` → faille XSS. Utiliser `textContent` pour du texte brut, ou échapper/sanitiser si du HTML est réellement nécessaire.

**Bonne pratique** : mettre en cache les références DOM récupérées plusieurs fois (éviter les `querySelector` répétés dans une boucle) ; regrouper les manipulations DOM pour limiter les reflows/repaints coûteux.

### 16 - Events

**Explication** — Le navigateur émet des événements (clic, saisie, chargement...) auxquels on s'abonne avec `addEventListener`.

```js
button.addEventListener("click", (event) => {
  event.preventDefault();  // empêche le comportement par défaut (ex. soumission de formulaire)
  console.log(event.target); // élément qui a déclenché l'événement
});

// Délégation d'événements — un seul listener sur le parent
list.addEventListener("click", (event) => {
  if (event.target.matches(".item")) {
    handleItemClick(event.target);
  }
});
```

**Bubbling** : un événement remonte du plus profond vers ses ancêtres (par défaut). **Capturing** : phase descendante, activable via `{ capture: true }`.

**Erreur fréquente** : attacher un listener à chaque élément d'une liste dynamique (fuite mémoire potentielle, coûteux à l'ajout/suppression) au lieu d'utiliser la délégation sur le conteneur parent.

**Bonne pratique** : toujours retirer un listener (`removeEventListener`) quand l'élément est détruit dynamiquement, sauf s'il l'est aussi (sinon fuite mémoire) ; préférer la délégation pour des listes dynamiques.

### 17 - Fetch / API

**Explication** — `fetch()` est l'API native pour faire des requêtes HTTP, basée sur les Promises.

```js
async function getUsers() {
  const response = await fetch("https://api.example.com/users");
  if (!response.ok) {
    throw new Error(`Erreur HTTP ${response.status}`);
  }
  return response.json();
}

async function createUser(data) {
  const response = await fetch("https://api.example.com/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  return response.json();
}
```

**Erreur fréquente** : oublier que `fetch` ne rejette **pas** la promesse sur une erreur HTTP (404, 500) — seule une erreur réseau (connexion coupée) la rejette. Il faut vérifier `response.ok` manuellement.

**Bonne pratique** : centraliser les appels API dans un module dédié avec gestion d'erreur cohérente ; utiliser `AbortController` pour annuler une requête (ex. recherche instantanée avec debounce).

```js
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort(); // annule la requête en cours
```

## 4. Concepts intermédiaires 🟡

- **Closures avancées** : encapsulation d'état privé (pattern module), mémoïsation, factory functions. Piège classique déjà vu au 06 (boucle + `var` + `setTimeout`).
- **`this` et son binding** : dépend de *comment* la fonction est appelée, pas d'où elle est définie (sauf arrow function). Quatre règles : appel simple (`this` = `undefined` en mode strict / `window` sinon), appel de méthode (`this` = l'objet), `call`/`apply`/`bind` (binding explicite), `new` (nouvel objet).

```js
function show() { console.log(this.name); }
const obj = { name: "A", show };
obj.show();               // "A" — appel de méthode
const detached = obj.show;
detached();                 // undefined — appel simple, contexte perdu
detached.call(obj);         // "A" — binding explicite
const bound = detached.bind(obj);
bound();                     // "A" — binding permanent
```

- **Prototypes et héritage prototypal** : chaque objet a un prototype (`Object.getPrototypeOf`), chaîne consultée à la résolution d'une propriété absente. Les classes ES6 sont du sucre syntaxique par-dessus ce mécanisme.
- **Event loop / microtasks vs macrotasks** : JS est mono-thread. Le call stack exécute le code synchrone ; une fois vide, la **microtask queue** (Promises, `queueMicrotask`) est entièrement vidée avant chaque **macrotask** (`setTimeout`, événements I/O). Explique pourquoi un `Promise.resolve().then()` s'exécute avant un `setTimeout(fn, 0)`.
- **Modules ESM avancé** : imports dynamiques (`const mod = await import("./mod.js")`, code-splitting/lazy loading), tree-shaking (le bundler élimine les exports non utilisés — nécessite des exports statiquement analysables).
- **Gestion d'erreurs avancée** : erreurs custom hiérarchisées, pattern "Result" (retourner `{ ok, value/error }` plutôt que throw) pour un flux de contrôle explicite dans du code métier critique.
- **Immutabilité** : `Object.freeze()` (superficiel, ne gèle pas les objets imbriqués), structures immuables par convention (spread systématique), intérêt pour la prévisibilité et la détection de changement (frameworks réactifs).
- **Performance courante** : `debounce` (attendre une pause avant d'exécuter, ex. recherche live) vs `throttle` (limiter la fréquence, ex. scroll) ; fuites mémoire fréquentes : listeners non retirés, closures retenant de gros objets, timers non nettoyés (`clearInterval`).
- **Debugging** : `debugger` + DevTools, `console.table`/`console.group`, breakpoints conditionnels, Network tab pour les appels `fetch`.

## 5. Concepts avancés 🟠🔴

- **Event loop en détail** : ordre exact microtask/macrotask/render, `requestAnimationFrame` pour les animations synchronisées au rafraîchissement écran, starvation de macrotasks par une chaîne de microtasks infinie.
- **Concurrence réelle** : Web Workers pour du calcul lourd hors thread principal (pas de partage direct de mémoire, communication par messages `postMessage`) ; `SharedArrayBuffer`/Atomics pour les cas avancés de mémoire partagée.
- **Design patterns JS** : Module, Observer/PubSub, Singleton (souvent anti-pattern en JS moderne), Factory, Proxy (`new Proxy()` pour intercepter get/set — base de la réactivité de Vue 3).
- **Optimisation moteur (V8, haut niveau)** : hidden classes / shape des objets (garder une forme d'objet stable améliore les perfs), éviter de changer le type d'une propriété après création, coût des `try/catch` en boucle chaude (moins vrai sur V8 récent mais reste une bonne pratique de lisibilité).
- **Sécurité** : XSS (déjà vu au DOM), injection via `eval`/`new Function` (à bannir), `Content-Security-Policy`, ne jamais faire confiance à une donnée venant du client.
- **Maintenabilité à grande échelle** : architecture en couches (séparation logique métier / accès données / UI), principes SOLID appliqués en JS (voir [`../design-patterns/`](../design-patterns/)), typage progressif via JSDoc ou migration TypeScript.
- **Scalabilité** : code-splitting, lazy loading de modules, pagination/virtualisation de listes longues côté DOM.

## 6. Commandes / syntaxe à connaître

```js
// Exécution rapide
node script.js                 // exécuter un fichier JS avec Node.js

// Console (debug)
console.log(value);
console.table(arrayOfObjects);
console.error(err);
console.time("label"); /* ... */ console.timeEnd("label");

// Syntaxe essentielle à avoir sous les doigts
const { a, b = 1, ...rest } = obj;   // destructuring + défaut + rest
const arr2 = [...arr1, x];            // spread
arr.map(x => x * 2).filter(x => x > 2); // chaînage fonctionnel
const result = await Promise.all([p1, p2]);
try { /* ... */ } catch (e) { /* ... */ } finally { /* ... */ }
export default function () {}
import x, { y } from "./mod.js";
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Todo-list avec persistance et API simulée**

Construire une todo-list en JS vanilla (pas de framework) qui doit :
- Ajouter/supprimer/marquer une tâche comme faite via manipulation du DOM et délégation d'événements.
- Persister les tâches dans `localStorage` (sérialisation JSON).
- Simuler un appel réseau asynchrone (`fetchTasks()` avec un `setTimeout` qui résout une Promise) pour charger les tâches au démarrage, avec gestion d'un état "chargement" et d'un état d'erreur.
- Utiliser `async/await`, `try/catch`, destructuring, spread, closures (compteur de tâches encapsulé), et au moins une fonction `debounce` (ex. sur un champ de filtre/recherche des tâches).

Objectif : mobiliser la quasi-totalité des notions du dossier dans un exercice concret et de taille raisonnable (quelques heures).

## Checklist

- [ ] Comprendre les fondamentaux (variables, types, scope, fonctions)
- [ ] Savoir créer un projet JS (fichier, module ESM, exécution navigateur/Node)
- [ ] Maîtriser la syntaxe principale (arrays, objects, destructuring, spread/rest)
- [ ] Comprendre les concepts importants (closures, `this`, event loop, prototypes)
- [ ] Savoir debugger (DevTools, breakpoints, console avancée)
- [ ] Connaître les bonnes pratiques (immutabilité, gestion d'erreurs, sécurité DOM)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Web Workers, optimisation V8, design patterns)

## 10. Ressources

- [MDN Web Docs — JavaScript](https://developer.mozilla.org/fr/docs/Web/JavaScript) — référence officielle et la plus fiable.
- [ECMAScript specification (TC39)](https://tc39.es/ecma262/) — spécification du langage, pour les questions de sémantique précise.
- [MDN — Event loop](https://developer.mozilla.org/fr/docs/Web/JavaScript/Event_loop) — référence pour la concurrence en JS.
- [javascript.info](https://javascript.info/) — tutoriel très complet, bon complément à MDN.
- [roadmap.sh — JavaScript](https://roadmap.sh/javascript) — vue d'ensemble du parcours d'apprentissage.
