# Exercices JavaScript — Niveau 1 : Bases

Objectif : vérifier la compréhension des fondamentaux (variables, types, conditions, boucles, fonctions, arrays, objects).

## 1. Variables et types

Prédire le résultat de chaque ligne SANS l'exécuter, puis vérifier :

```js
console.log(typeof "5" + 5);
console.log("5" + 5);
console.log("5" - 5);
console.log(1 == "1");
console.log(1 === "1");
console.log(null == undefined);
console.log(null === undefined);
```

## 2. Conditions

Écrire une fonction `getDiscount(amount)` qui retourne :
- `0.1` si `amount >= 100`
- `0.05` si `amount >= 50`
- `0` sinon

## 3. Boucles

Écrire une fonction `sumEvens(numbers)` qui retourne la somme des nombres pairs d'un tableau, en utilisant une boucle `for...of`.

## 4. Fonctions

Écrire une fonction `greet(name, greeting = "Bonjour")` qui retourne `"${greeting}, ${name} !"`, avec une valeur par défaut pour `greeting`.

## 5. Arrays

Étant donné `const prices = [10, 25, 8, 42, 15];`, sans boucle manuelle (`for`/`while`) :
1. Créer un nouveau tableau avec chaque prix +10% de TVA.
2. Filtrer les prix supérieurs à 20.
3. Calculer la somme totale des prix.

## 6. Objects

Étant donné :

```js
const product = { name: "Clavier", price: 49.99, inStock: true };
```

1. Afficher toutes les clés de l'objet.
2. Afficher toutes les valeurs de l'objet.
3. Créer une copie de `product` avec `price` à `39.99` sans muter l'original.

## 7. Destructuring

Étant donné `const point = { x: 10, y: 20, z: 30 };`, extraire `x` et `y` dans deux variables en une seule ligne.

## 8. Scope

Expliquer par écrit pourquoi ce code affiche `3, 3, 3` et proposer une correction affichant `0, 1, 2` :

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
