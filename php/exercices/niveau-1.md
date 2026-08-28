# Exercices PHP — Niveau 1 : Bases

Objectif : vérifier la compréhension des fondamentaux (types, opérateurs, structures de contrôle, tableaux, chaînes).

## 1. Types et comparaison

Prédire le résultat de chaque ligne SANS l'exécuter, puis vérifier avec `var_dump()` :

```php
var_dump(0 == "abc");
var_dump("1" == "01");
var_dump("1" === "01");
var_dump("10" == "1e1");
var_dump(null == false);
var_dump(null === false);
```

## 2. Conditions

Écrire une fonction `getShippingCost(float $amount): float` qui retourne :
- `0.0` si `$amount >= 100`
- `4.99` si `$amount >= 50`
- `9.99` sinon

## 3. Boucles et tableaux

Écrire une fonction `sumEvens(array $numbers): int` qui retourne la somme des nombres pairs d'un tableau, avec une boucle `foreach`.

## 4. Fonctions

Écrire une fonction `greet(string $name, string $greeting = "Bonjour"): string` qui retourne `"$greeting, $name !"`, typée et avec valeur par défaut.

## 5. Tableaux fonctionnels

Étant donné `$prices = [10, 25, 8, 42, 15];`, sans boucle manuelle (`for`/`while`/`foreach`) :
1. Créer un nouveau tableau avec chaque prix +20% de TVA (`array_map`).
2. Filtrer les prix supérieurs à 20 (`array_filter`).
3. Calculer la somme totale (`array_reduce` ou `array_sum`).

## 6. Chaînes de caractères

Étant donné `$email = "max@example.com";`, écrire une fonction `maskEmail(string $email): string` qui masque la partie avant le `@` sauf le premier caractère (ex: `m**@example.com`), en utilisant `str_contains`, `explode`, et `str_repeat`.

## 7. Tableaux associatifs

Étant donné :

```php
$product = ["name" => "Clavier", "price" => 49.99, "inStock" => true];
```

1. Vérifier si la clé `"discount"` existe avec `array_key_exists`.
2. Créer une copie de `$product` avec `price` à `39.99` sans muter l'original.
3. Expliquer par écrit pourquoi `isset($product["inStock"])` et `array_key_exists("inStock", $product)` peuvent donner des résultats différents dans certains cas (donner un exemple).
