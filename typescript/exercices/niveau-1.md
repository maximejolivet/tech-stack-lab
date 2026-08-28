# Exercices TypeScript — Niveau 1 : Bases

Objectif : vérifier la compréhension des fondamentaux (types, interfaces, unions, narrowing, generics simples).

## 1. Typer une fonction

Ajouter les annotations de type manquantes à cette fonction (paramètres et retour) :

```ts
function calculateTotal(price, quantity, taxRate) {
  return price * quantity * (1 + taxRate);
}
```

## 2. Interface

Définir une interface `Product` avec : `id` (number), `name` (string), `price` (number), `description` (optionnel, string), `inStock` (boolean, en lecture seule).

## 3. Union de littéraux

Définir un type `OrderStatus` limité aux valeurs `"pending"`, `"shipped"`, `"delivered"`, `"cancelled"`. Écrire une fonction `isFinal(status: OrderStatus): boolean` qui retourne `true` seulement pour `"delivered"` et `"cancelled"`.

## 4. Narrowing

Écrire une fonction `formatValue(value: string | number | boolean): string` qui :
- pour une `string`, la retourne en majuscules
- pour un `number`, la retourne avec 2 décimales
- pour un `boolean`, retourne `"oui"` ou `"non"`

## 5. `any` vs `unknown`

Ce code compile mais crashe au runtime. Corriger en utilisant `unknown` et un narrowing approprié, sans changer le comportement attendu :

```ts
function processInput(input: any) {
  return input.toUpperCase();
}
```

## 6. Generic simple

Écrire une fonction générique `wrapInArray<T>(value: T): T[]` qui retourne un tableau contenant uniquement la valeur passée, en préservant son type.

## 7. Utility type

Étant donné l'interface `Product` de l'exercice 2, définir un type `ProductPreview` qui ne garde que `id` et `name` (sans dupliquer la définition), et un type `ProductUpdate` qui rend toutes les propriétés optionnelles.
