# Solutions PHP — Niveau 1

## 1. Types et comparaison

```php
var_dump(0 == "abc");     // false — PHP 8 compare "abc" comme une chaîne (0 n'est plus égal à une chaîne non-numérique)
var_dump("1" == "01");    // true  — les deux chaînes sont numériques, comparées comme des nombres (1 == 1)
var_dump("1" === "01");   // false — types identiques (string) mais valeurs différentes
var_dump("10" == "1e1");  // true  — "1e1" est numérique (notation scientifique = 10), comparaison numérique
var_dump(null == false);  // true  — null est falsy, comparaison loose
var_dump(null === false); // false — types différents (NULL vs bool)
```

## 2. Conditions

```php
function getShippingCost(float $amount): float
{
    if ($amount >= 100) {
        return 0.0;
    }
    if ($amount >= 50) {
        return 4.99;
    }
    return 9.99;
}
```

## 3. Boucles et tableaux

```php
function sumEvens(array $numbers): int
{
    $sum = 0;
    foreach ($numbers as $n) {
        if ($n % 2 === 0) {
            $sum += $n;
        }
    }
    return $sum;
}
```

## 4. Fonctions

```php
function greet(string $name, string $greeting = "Bonjour"): string
{
    return "$greeting, $name !";
}
```

## 5. Tableaux fonctionnels

```php
$prices = [10, 25, 8, 42, 15];

$withTax = array_map(fn($p) => $p * 1.2, $prices);
$expensive = array_filter($prices, fn($p) => $p > 20);
$total = array_sum($prices); // ou array_reduce($prices, fn($c, $p) => $c + $p, 0)
```

## 6. Chaînes de caractères

```php
function maskEmail(string $email): string
{
    if (!str_contains($email, "@")) {
        return $email;
    }

    [$local, $domain] = explode("@", $email, 2);
    $masked = substr($local, 0, 1) . str_repeat("*", max(strlen($local) - 1, 0));

    return "$masked@$domain";
}

maskEmail("max@example.com"); // "m**@example.com"
```

## 7. Tableaux associatifs

```php
$product = ["name" => "Clavier", "price" => 49.99, "inStock" => true];

var_dump(array_key_exists("discount", $product)); // false

$discounted = [...$product, "price" => 39.99]; // copie + override (PHP 8.1+ spread préserve les clés string)
// équivalent plus explicite : $discounted = $product; $discounted["price"] = 39.99;
```

**Point 3** : `isset()` retourne `false` si la valeur associée à la clé est `null`, même si la clé existe. `array_key_exists()` retourne `true` dès que la clé existe, peu importe sa valeur.

```php
$data = ["inStock" => null];
isset($data["inStock"]);              // false — la valeur est null
array_key_exists("inStock", $data);   // true  — la clé existe bel et bien
```
