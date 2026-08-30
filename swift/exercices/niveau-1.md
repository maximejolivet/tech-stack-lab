# Niveau 1 — Bases

## Exercice 1.1 — Hello, typé

Écris une petite fonction `main` (ou un simple script top-level) qui déclare un `let name: String`, un `let age: Int`, et affiche `"Bonjour <name>, tu as <age> ans."` avec une interpolation de chaîne (pas de concaténation `+`).

## Exercice 1.2 — Optionals

Sans exécuter le code, prédis le résultat de chaque ligne, puis vérifie :

```swift
var nickname: String? = nil

print(nickname?.count)
print(nickname?.count ?? -1)
print(nickname?.uppercased()?.count ?? 0)
```

Explique en une phrase la différence entre `?.` et `??`.

## Exercice 1.3 — Array immuable vs mutable

Déclare un `let scores = [12, 15, 8, 18, 10]`. Tente d'appeler `.append()` dessus et observe l'erreur de compilation. Corrige en déclarant la variable avec `var`, puis calcule la moyenne des scores (indice : `.reduce(0, +)` puis division par `.count`).

## Exercice 1.4 — Switch exhaustif

Écris une fonction `func dayType(_ day: Int) -> String` qui retourne `"Week-end"` pour 6 et 7, `"Semaine"` pour 1 à 5, `"Invalide"` sinon, en utilisant un `switch` avec des intervalles (`1...5`).

## Exercice 1.5 — Struct simple

Déclare une `struct Point { var x: Int; var y: Int }`. Crée une instance `p1`, copie-la dans `p2`, modifie `p2.x`, puis affiche `p1.x` et `p2.x` pour vérifier qu'ils sont bien indépendants.
