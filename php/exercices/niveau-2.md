# Exercices PHP — Niveau 2 : Intermédiaire

Objectif : mobiliser POO, exceptions, namespaces/autoload et manipulation de JSON.

## 1. Classe avec property promotion

Créer une classe `Product` avec :
- Propriétés `readonly` : `name` (string), `price` (float).
- Un constructeur utilisant la property promotion (PHP 8).
- Une méthode `priceWithTax(float $rate): float` qui retourne le prix TTC.
- Vérifier qu'une tentative de modifier `price` après création lève bien une erreur (propriété `readonly`).

## 2. Héritage et interfaces

Créer une interface `Discountable` avec une méthode `applyDiscount(float $percent): float`.
Créer deux classes `Product` et `Service` qui l'implémentent chacune différemment (`Product` réduit le prix, `Service` a un minimum non réductible en dessous d'un seuil).
Écrire une fonction `totalAfterDiscount(array $items, float $percent): float` qui accepte un tableau d'objets `Discountable` (peu importe leur classe concrète) et retourne la somme après réduction.

## 3. Exceptions métier

Créer une exception `InsufficientStockException extends \RuntimeException` avec un constructeur prenant le nom du produit et la quantité demandée, formatant un message clair.
Écrire une fonction `reserveStock(string $product, int $requested, int $available): void` qui lève cette exception si `$requested > $available`, et l'appeler dans un bloc `try/catch/finally` qui logue systématiquement (même en cas de succès) un message dans le `finally`.

## 4. JSON et tableaux associatifs

Étant donné une chaîne JSON représentant une liste de commandes :

```php
$json = '[{"id":1,"total":49.99,"status":"paid"},{"id":2,"total":120.00,"status":"pending"}]';
```

1. Décoder en tableau associatif (pas en `stdClass`).
2. Calculer la somme des `total` des commandes dont `status` vaut `"paid"`.
3. Ré-encoder uniquement les commandes `"pending"` en JSON.

## 5. Trait

Créer un trait `Timestampable` avec deux propriétés (`createdAt`, `updatedAt` en `\DateTimeImmutable`) et une méthode `touch()` qui met à jour `updatedAt`. L'utiliser dans une classe `Article` sans passer par l'héritage.
