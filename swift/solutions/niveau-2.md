# Solutions — Niveau 2

## 2.1 — Enum à valeurs associées

```swift
enum Shape {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
}

func area(_ shape: Shape) -> Double {
    switch shape {
    case .circle(let radius): return .pi * radius * radius
    case .rectangle(let width, let height): return width * height
    // pas de "default" : le compilateur refuse de compiler si un cas de l'enum manque
    }
}
```

## 2.2 — Vue SwiftUI avec @State

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        Button("Compteur : \(count)") {
            count += 1 // modifie @State → SwiftUI redessine automatiquement la vue
        }
    }
}
```

## 2.3 — Binding entre vues

```swift
struct ToggleView: View {
    @Binding var isOn: Bool
    var body: some View {
        Toggle("Actif", isOn: $isOn)
    }
}

struct ParentView: View {
    @State private var isOn = false

    var body: some View {
        VStack {
            Text(isOn ? "Activé" : "Désactivé")
            ToggleView(isOn: $isOn) // $ crée le Binding depuis le @State du parent
        }
    }
}
```

## 2.4 — ObservableObject

```swift
class CartModel: ObservableObject {
    @Published var items: [String] = []
    func add(_ item: String) { items.append(item) } // modifie @Published → notifie tous les observateurs
}

struct CartView: View {
    @StateObject var cart = CartModel()

    var body: some View {
        VStack {
            Text("\(cart.items.count) articles")
            Button("Ajouter") { cart.add("Nouvel article") }
        }
    }
}
```

## 2.5 — Async/await

```swift
func fetchUserName() async -> String {
    try? await Task.sleep(nanoseconds: 500_000_000) // simule 500ms de latence réseau
    return "Max"
}

struct UserView: View {
    @State private var name = "Chargement..."

    var body: some View {
        Text(name).task {
            name = await fetchUserName() // exécuté dès l'apparition de la vue
        }
    }
}
```
