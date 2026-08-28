# Solutions — Niveau 1 : Bases

## 1. Variables et types

```js
console.log(typeof "5" + 5);   // "string5" — typeof "5" vaut "string", puis "string" + 5 concatène
console.log("5" + 5);           // "55"      — + avec une string = concaténation
console.log("5" - 5);           // 0         — - force la conversion numérique
console.log(1 == "1");          // true      — coercition de type
console.log(1 === "1");         // false     — types différents, pas de coercition
console.log(null == undefined); // true      — cas spécial de ==
console.log(null === undefined);// false     — types différents
```

## 2. Conditions

```js
function getDiscount(amount) {
  if (amount >= 100) return 0.1;
  if (amount >= 50) return 0.05;
  return 0;
}
```

## 3. Boucles

```js
function sumEvens(numbers) {
  let total = 0;
  for (const n of numbers) {
    if (n % 2 === 0) total += n;
  }
  return total;
}
```

## 4. Fonctions

```js
function greet(name, greeting = "Bonjour") {
  return `${greeting}, ${name} !`;
}
```

## 5. Arrays

```js
const prices = [10, 25, 8, 42, 15];

const withTax = prices.map(p => p * 1.1);
const expensive = prices.filter(p => p > 20);
const total = prices.reduce((sum, p) => sum + p, 0);
```

## 6. Objects

```js
const product = { name: "Clavier", price: 49.99, inStock: true };

console.log(Object.keys(product));
console.log(Object.values(product));
const onSale = { ...product, price: 39.99 };
```

## 7. Destructuring

```js
const point = { x: 10, y: 20, z: 30 };
const { x, y } = point;
```

## 8. Scope

`var` a un scope de **fonction**, pas de bloc : une seule variable `i` est partagée par toutes les itérations de la boucle. Au moment où les callbacks `setTimeout` s'exécutent (après la boucle), `i` vaut déjà `3`.

Correction — remplacer `var` par `let`, qui crée une nouvelle liaison de `i` à **chaque itération** (scope de bloc) :

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0, 1, 2
}
```
