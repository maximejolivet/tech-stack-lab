# Swift

## 1. Introduction

Swift est le langage natif d'Apple pour construire des applications iOS, macOS, watchOS et tvOS, compilé en code machine natif (pas de VM ni de pont, contrairement à React Native [`../react-native/`](../react-native/) ou au moteur de rendu embarqué de Flutter [`../flutter/`](../flutter/)). Ce dossier couvre Swift et **SwiftUI** (framework déclaratif moderne pour construire l'UI) ensemble, comme Flutter couvre Dart et son framework dans un seul dossier.

**À quoi sert-il ?**
- Construire des applications natives iOS/macOS avec un accès direct et complet à toutes les API de la plateforme, sans couche d'abstraction intermédiaire.
- Obtenir la meilleure performance et la meilleure intégration système possibles sur l'écosystème Apple (widgets iOS, Live Activities, intégration profonde avec les capteurs).

**Où se situe-t-il dans une architecture mobile ?** Swift compile directement en code machine ARM64 natif, exécuté sans interprète ni pont — SwiftUI décrit l'UI déclarativement (comme React/Flutter) mais la traduit en composants UIKit/AppKit natifs sous le capot, contrairement à Flutter qui dessine lui-même chaque pixel.

**Avantages**
- Accès natif complet et immédiat aux nouveautés Apple (une nouvelle API iOS est disponible en Swift dès la sortie, avant tout wrapper cross-platform).
- Performance native maximale : pas de pont JS (React Native) ni de moteur de rendu embarqué (Flutter) à faire tourner.
- Typage statique très strict avec inférence, `Optional` intégré au système de types — sécurité proche de Kotlin ([`../kotlin/`](../kotlin/)) côté élimination des erreurs de nullité.

**Limites**
- Limité à l'écosystème Apple : aucune app Android produite depuis ce code (contrairement à React Native/Flutter qui ciblent les deux plateformes depuis une seule base).
- Nécessite un Mac pour développer et compiler (Xcode n'existe que sur macOS) — contrainte matérielle absente des autres approches mobiles de cette roadmap.
- Écosystème et outillage (Xcode, SwiftUI Previews) parfois moins stables que des outils web/JS matures.

## 2. Prérequis

- POO déjà pratiquée dans un autre langage (PHP, Java, Kotlin) — classes, protocoles (interfaces), héritage sont directement transférables.
- Mac avec Xcode installé (gratuit sur le Mac App Store) — seul environnement de développement iOS officiellement supporté.
- Aucun prérequis JavaScript nécessaire, comme pour Flutter/Dart — Swift est un langage à part entière, couvert en section 3.

## 3. Rappel des bases 🟢

### 01 - Syntaxe et typage

**Explication** — Swift est un langage compilé, à **typage statique fort** avec inférence de type quasi systématique — proche de Kotlin dans sa philosophie (`let` immuable par défaut, `var` mutable).

```swift
let name = "Max"        // immuable, type inféré String
var age = 30              // mutable, type inféré Int
let city: String = "Paris"  // typage explicite (rarement nécessaire, inféré la plupart du temps)

print("Bonjour \(name), tu as \(age) ans.")  // interpolation avec \(...)
```

**Bonne pratique** : comme en Kotlin, toujours commencer par `let`, ne passer à `var` que si la mutabilité est réellement nécessaire — limite les bugs d'état partagé.

### 02 - Optionals (null safety)

**Explication** — `Optional<T>` (sucre syntaxique `T?`) représente une valeur qui peut être absente — équivalent direct du `T?` de Kotlin ([`../kotlin/`](../kotlin/)) ou du `String?` de Dart. Le compilateur force à "déballer" un optional avant de l'utiliser.

```swift
var nickname: String? = nil

if let unwrapped = nickname {           // optional binding : ne s'exécute que si non-nil
    print("Pseudo : \(unwrapped)")
}

print(nickname?.count)                    // optional chaining : nil si nickname est nil
print(nickname ?? "Anonyme")               // nil-coalescing : valeur par défaut si nil
print(nickname!.count)                       // force-unwrap : crash si nil, à éviter
```

**Erreur fréquente** : abuser du force-unwrap (`!`) pour faire taire le compilateur — provoque un crash immédiat de l'app si la valeur est réellement `nil`, exactement le risque que les optionals visent à éliminer (même piège que `!!` en Kotlin).

### 03 - Structures de contrôle

**Explication** — `if/else`, `for-in`, `while` proches des autres langages ; `switch` est **exhaustif par défaut** (le compilateur exige de couvrir tous les cas ou d'avoir un `default`), et matche des valeurs, des intervalles, voire des tuples.

```swift
let age = 20
let status = age >= 18 ? "majeur" : "mineur"   // opérateur ternaire

switch age {
case 0..<13: print("Enfant")
case 13..<18: print("Adolescent")
default: print("Adulte")
}
```

**Cas d'usage** : le `switch` exhaustif sur une `enum` (voir section 05) est l'idiome central de la gestion d'état en Swift/SwiftUI, comparable au `when` exhaustif de Kotlin sur une `sealed class`.

### 04 - Collections

**Explication** — `Array`, `Dictionary`, `Set` sont les collections natives ; **immuables par défaut** avec `let`, mutables avec `var` — le même principe qu'en Kotlin (`listOf` vs `mutableListOf`), ici porté directement par le mot-clé de déclaration plutôt que par le type.

```swift
let scores = [12, 15, 8, 18, 10]           // Array immuable (let)
var editableScores = [12, 15, 8]            // Array mutable (var)
editableScores.append(20)

let ages: [String: Int] = ["Max": 30, "Alice": 25]  // Dictionary
print(ages["Bob"] ?? 0)                                // valeur par défaut si clé absente
```

### 05 - Enums avec valeurs associées

**Explication** — Une `enum` Swift peut porter des données associées à chaque cas (bien plus riche qu'une énumération Java/PHP classique) — usage très proche d'une `sealed class` Kotlin pour modéliser un état fini avec des données variables selon le cas.

```swift
enum PaymentEvent {
    case succeeded(orderId: String, amount: Double)
    case failed(orderId: String, reason: String)
}

func describe(_ event: PaymentEvent) -> String {
    switch event {
    case .succeeded(let orderId, let amount):
        return "Commande \(orderId) payée : \(amount)€"
    case .failed(let orderId, let reason):
        return "Commande \(orderId) échouée : \(reason)"
    }
}
```

**Bonne pratique** : modéliser un état métier fini (statut de commande, résultat réseau) avec une `enum` à valeurs associées plutôt qu'avec plusieurs booléens/champs optionnels combinés — élimine les combinaisons d'état invalides.

### 06 - Structs vs classes

**Explication** — Swift distingue explicitement `struct` (type valeur, copié à chaque affectation) et `class` (type référence, partagé par pointeur) — SwiftUI encourage fortement les `struct` pour les modèles de données et les vues elles-mêmes.

```swift
struct Point {
    var x: Int
    var y: Int
}

var p1 = Point(x: 1, y: 2)
var p2 = p1          // copie complète, indépendante
p2.x = 99
print(p1.x)          // 1 — p1 n'est pas affecté, contrairement à une class
```

**Erreur fréquente** : s'attendre à un comportement partagé (référence) sur une `struct` par réflexe venant d'un langage où tout objet est une référence (Java, PHP, JS) — une `struct` Swift copiée n'affecte jamais l'original.

### 07 - Vue SwiftUI minimale

**Explication** — Une vue SwiftUI est une `struct` conforme au protocole `View`, décrivant déclarativement l'UI dans sa propriété `body` — même philosophie déclarative que React/Flutter, sans langage de template séparé (comme Flutter, contrairement à Vue/Angular).

```swift
struct GreetingView: View {
    let name: String

    var body: some View {
        Text("Bonjour, \(name) !")
            .font(.title)
            .padding()
    }
}
```

**Cas d'usage** : `some View` masque le type concret réellement retourné (souvent un type imbriqué complexe généré par le compilateur) — il suffit de savoir que la vue "se comporte comme une View", sans connaître son type exact.

## 4. Concepts intermédiaires 🟡

- **State et gestion d'état local** : `@State` marque une propriété comme source de vérité locale à une vue — modifier sa valeur redessine automatiquement la vue, équivalent direct de `setState`/`useState` en Flutter/React.

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        Button("Compteur : \(count)") {
            count += 1   // modifie @State → redessine automatiquement la vue
        }
    }
}
```

- **Binding et flux de données entre vues** : `@Binding` permet à une vue enfant de lire et modifier un `@State` détenu par une vue parente, sans dupliquer l'état — équivalent conceptuel des props + callback en React, mais avec une syntaxe dédiée à double sens.

```swift
struct ToggleView: View {
    @Binding var isOn: Bool
    var body: some View { Toggle("Actif", isOn: $isOn) }  // $ crée le Binding depuis un @State
}
```

- **ObservableObject et @StateObject** : pour un état partagé entre plusieurs vues éloignées dans la hiérarchie (équivalent du `ChangeNotifier` + `Provider` de Flutter), une classe conforme à `ObservableObject` expose des propriétés `@Published` qui déclenchent un redessin de toute vue observatrice.

```swift
class CartModel: ObservableObject {
    @Published var items: [String] = []
    func add(_ item: String) { items.append(item) }
}

struct CartView: View {
    @StateObject var cart = CartModel()
    var body: some View { Text("\(cart.items.count) articles") }
}
```

- **Programmation asynchrone (async/await)** : syntaxe structurée pour les appels asynchrones, très proche des coroutines Kotlin ([`../kotlin/`](../kotlin/)) et de `async`/`await` en Dart/JS.

```swift
func fetchUserName() async throws -> String {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data).name
}
```

- **Navigation avec NavigationStack** : pile de navigation déclarative (iOS 16+), comparable au `Navigator` de Flutter ou à React Navigation, intégrée nativement au framework plutôt qu'une librairie tierce.

```swift
NavigationStack {
    List(items) { item in
        NavigationLink(item.title, value: item.id)
    }
    .navigationDestination(for: Int.self) { id in DetailView(id: id) }
}
```

## 5. Concepts avancés 🟠🔴

- **Protocol-oriented programming** : Swift encourage la composition via des `protocol` (interfaces) et des extensions par défaut (*protocol extensions*) plutôt que l'héritage de classes — un protocole peut fournir une implémentation par défaut de ses méthodes, réutilisable par tout type conforme sans hiérarchie de classes.

```swift
protocol Greetable {
    var name: String { get }
    func greet() -> String
}

extension Greetable {
    func greet() -> String { "Bonjour, \(name) !" }  // implémentation par défaut, héritée sans "extends"
}
```

- **Property wrappers custom** : `@State`, `@Published` sont eux-mêmes des *property wrappers* — un mécanisme réutilisable pour encapsuler un comportement (validation, persistance, notification de changement) autour d'une propriété, définissable soi-même avec `@propertyWrapper`.
- **Gestion mémoire ARC (Automatic Reference Counting)** : contrairement au garbage collector "stop-the-world" périodique de la JVM (Java/Kotlin), Swift compte les références à chaque objet et le libère dès que ce compteur atteint zéro — déterministe, mais nécessite de gérer explicitement les cycles de rétention (`weak`/`unowned`) entre objets qui se référencent mutuellement.

```swift
class Owner {
    var pet: Pet?
}
class Pet {
    weak var owner: Owner?   // weak casse le cycle de rétention Owner ↔ Pet
}
```

- **Concurrency structurée (actors)** : `actor` garantit qu'un seul bloc de code accède à son état interne à la fois, éliminant les *data races* à la compilation — équivalent conceptuel des coroutines confinées à un `Dispatcher` unique en Kotlin, mais imposé par le compilateur plutôt que par convention.
- **SwiftUI Previews et Xcode** : aperçu en temps réel de l'UI directement dans l'IDE sans lancer le simulateur complet — outillage propre à l'écosystème Xcode, sans équivalent direct côté React Native/Flutter (dont le hot reload nécessite l'app en cours d'exécution).

## 6. Commandes / syntaxe à connaître

```bash
xcodebuild -version              # vérifie la version d'Xcode installée
swift --version                   # vérifie la version du compilateur Swift

xcodebuild -scheme MonApp build     # build en ligne de commande
xcodebuild test -scheme MonApp        # lancer les tests en ligne de commande

swift package init                     # créer un nouveau Swift Package
swift build                              # build via Swift Package Manager
swift test                                 # lancer les tests via SPM
```

```swift
struct MyView: View {
    @State private var value = 0
    var body: some View {
        Text("Valeur : \(value)")
            .onTapGesture { value += 1 }
    }
}

if let unwrapped = optionalValue { /* ... */ }
let result = optionalValue ?? "défaut"
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Application de notes avec navigation et état partagé**

Reprendre le mini-projet Flutter ([`../flutter/README.md`](../flutter/README.md)) en Swift/SwiftUI, pour comparer directement les deux approches mobiles sur le même exercice :
- Une vue `NoteListView` affichant une liste de notes via `List`.
- Un `ObservableObject NotesModel` avec des propriétés `@Published`, exposant `addNote`/`removeNote`.
- Une vue `NoteFormView` avec un `Form` et des champs validés pour ajouter une note, poussée via `NavigationStack`.
- Chaque note de la liste navigue vers une vue de détail via `NavigationLink`.
- Bonus : charger une liste d'utilisateurs depuis une API publique avec `async`/`await` et `URLSession`, en affichant un `ProgressView` pendant le chargement.

Objectif : mobiliser `@State`/`@Binding`/`ObservableObject`, navigation, formulaires et concurrence structurée dans le même exercice concret que la version Flutter, pour ressentir la différence entre une UI qui pilote des composants natifs et une UI qui dessine elle-même chaque pixel.

## Checklist

- [ ] Comprendre les fondamentaux de Swift (typage, optionals, structs vs classes)
- [ ] Comprendre le modèle déclaratif de SwiftUI (vues, `body`, `some View`)
- [ ] Savoir créer un projet Xcode et utiliser les Previews
- [ ] Maîtriser la syntaxe principale (`@State`, `@Binding`, enums à valeurs associées)
- [ ] Comprendre les concepts importants (ObservableObject/@Published, async/await, NavigationStack)
- [ ] Savoir debugger avec Xcode (breakpoints, View Debugger)
- [ ] Connaître les bonnes pratiques (`let` par défaut, éviter le force-unwrap, `weak` pour casser les cycles ARC)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (protocol-oriented programming, ARC, actors)

## 10. Ressources

- [Documentation officielle Swift](https://www.swift.org/documentation/) — référence complète du langage.
- [Documentation officielle SwiftUI (Apple Developer)](https://developer.apple.com/documentation/swiftui/) — référence du framework UI.
- [Swift Playgrounds](https://developer.apple.com/swift-playgrounds/) — environnement interactif pour pratiquer la syntaxe.
- [Hacking with Swift](https://www.hackingwithswift.com/) — tutoriels et projets guidés de qualité.
- [roadmap.sh — iOS Developer](https://roadmap.sh/ios) — vue d'ensemble du parcours d'apprentissage.
