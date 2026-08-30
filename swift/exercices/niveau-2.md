# Niveau 2 — Intermédiaire

## Exercice 2.1 — Enum à valeurs associées

Modélise `enum Shape { case circle(radius: Double); case rectangle(width: Double, height: Double) }`. Écris une fonction `func area(_ shape: Shape) -> Double` avec un `switch` exhaustif (sans `default`) qui calcule l'aire selon le cas.

## Exercice 2.2 — Vue SwiftUI avec @State

Écris une `struct CounterView: View` avec un `@State private var count = 0` et un `Button` qui incrémente `count` à chaque tap, affichant sa valeur courante dans le libellé du bouton.

## Exercice 2.3 — Binding entre vues

Écris une vue parente `ParentView` possédant un `@State private var isOn = false`, qui passe un `Binding` à une vue enfant `ToggleView` (avec `@Binding var isOn: Bool` et un `Toggle`). Vérifie que basculer le toggle dans l'enfant met bien à jour l'état du parent.

## Exercice 2.4 — ObservableObject

Crée une classe `CartModel: ObservableObject` avec une propriété `@Published var items: [String] = []` et une méthode `func add(_ item: String)`. Utilise-la dans une vue via `@StateObject` et affiche le nombre d'articles, qui doit se mettre à jour automatiquement après un appel à `add`.

## Exercice 2.5 — Async/await

Écris une fonction `func fetchUserName() async -> String` qui simule une latence réseau (`try? await Task.sleep(nanoseconds: 500_000_000)`) puis retourne un nom. Appelle-la depuis une `Task { ... }` et affiche le résultat une fois disponible.
