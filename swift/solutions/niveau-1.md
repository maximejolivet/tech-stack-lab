# Solutions — Niveau 1

## 1.1 — Hello, typé

```swift
let name = "Max"
let age = 30
print("Bonjour \(name), tu as \(age) ans.")
```

## 1.2 — Optionals

```swift
print(nickname?.count)                          // nil (optional chaining sur nil → nil, pas de crash)
print(nickname?.count ?? -1)                      // -1 (nil-coalescing fournit la valeur par défaut)
print(nickname?.uppercased()?.count ?? 0)          // 0 (chaîne d'optional chaining, s'arrête au premier nil)
```

`?.` (optional chaining) propage `nil` immédiatement si l'objet est `nil` au lieu de crasher ; `??` (nil-coalescing) fournit une valeur de remplacement quand l'expression à sa gauche vaut `nil`.

## 1.3 — Array immuable vs mutable

```swift
let scores = [12, 15, 8, 18, 10]
// scores.append(20) // erreur de COMPILATION : append nécessite un Array mutable (var)

var editableScores = [12, 15, 8, 18, 10]
editableScores.append(20) // OK

let average = Double(scores.reduce(0, +)) / Double(scores.count) // 12.6
```

## 1.4 — Switch exhaustif

```swift
func dayType(_ day: Int) -> String {
    switch day {
    case 6, 7: return "Week-end"
    case 1...5: return "Semaine"
    default: return "Invalide"
    }
}
```

## 1.5 — Struct simple

```swift
struct Point {
    var x: Int
    var y: Int
}

var p1 = Point(x: 1, y: 2)
var p2 = p1        // copie complète, indépendante (type valeur)
p2.x = 99

print(p1.x) // 1 — p1 n'est pas affecté
print(p2.x) // 99
```
