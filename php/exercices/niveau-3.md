# Exercices PHP — Niveau 3 : Avancé

Objectif : problèmes proches d'un contexte professionnel — enums, générateurs, match, closures, performance mémoire.

## 1. Enum avec logique métier

Créer un enum `OrderStatus: string` avec les cas `Pending`, `Paid`, `Shipped`, `Cancelled`.
Ajouter une méthode `canTransitionTo(OrderStatus $next): bool` qui encode les transitions valides (ex: `Pending` → `Paid` ou `Cancelled` uniquement ; `Paid` → `Shipped` ou `Cancelled` ; `Shipped` et `Cancelled` sont terminaux).
Utiliser `match` en interne pour implémenter cette logique. Écrire un test manuel qui vérifie qu'une transition invalide (`Shipped` → `Pending`) est bien rejetée.

## 2. Générateur pour un gros fichier

Un fichier `orders.csv` contient potentiellement plusieurs millions de lignes (`id,total,status`).
Écrire une fonction `readOrders(string $path): \Generator` qui lit le fichier ligne par ligne et `yield` un tableau associatif par ligne (`['id' => ..., 'total' => ..., 'status' => ...]`), sans jamais charger tout le fichier en mémoire.
Écrire une fonction `sumPaidTotal(iterable $orders): float` qui consomme ce générateur (ou n'importe quel `iterable`) pour sommer les commandes `"paid"`.
Expliquer par écrit pourquoi cette approche garde une mémoire constante quelle que soit la taille du fichier, contrairement à `file()` + `array_filter()`.

## 3. Closures et composition de fonctions

Écrire une fonction `pipe(callable ...$fns): callable` qui retourne une nouvelle fonction appliquant chaque fonction du tableau dans l'ordre, le résultat de l'une servant d'entrée à la suivante (composition fonctionnelle).

```php
$process = pipe(
    fn(int $x) => $x + 1,
    fn(int $x) => $x * 2,
    fn(int $x) => $x - 3,
);
$process(5); // ((5 + 1) * 2) - 3 = 9
```

Utiliser cette fonction pour construire un pipeline de normalisation de chaîne (trim → lowercase → suppression des espaces multiples).

## 4. Design orienté interface (mini refactor)

Voici un code volontairement mal structuré :

```php
class OrderProcessor
{
    public function process(array $order): void
    {
        if ($order['payment_method'] === 'card') {
            // logique de paiement carte, 15 lignes
        } elseif ($order['payment_method'] === 'paypal') {
            // logique de paiement paypal, 20 lignes
        } elseif ($order['payment_method'] === 'transfer') {
            // logique de paiement virement, 10 lignes
        }
    }
}
```

Refactorer ce code en définissant une interface `PaymentMethod` avec une méthode `pay(array $order): void`, une classe par mode de paiement, et une manière de choisir la bonne implémentation sans `if/elseif` en cascade dans `OrderProcessor` (indice : une `array` de mapping, ou un enum avec `match`). Justifier en une phrase le gain (ouvert à l'extension, fermé à la modification — principe SOLID, voir [`../design-patterns/`](../design-patterns/)).
