# Solutions — Niveau 3

## 3.1 — Protocol-oriented programming

```swift
protocol Payable {
    var amount: Double { get }
    func describe() -> String
}

extension Payable {
    func describe() -> String { "Montant : \(amount)€" } // implémentation par défaut
}

struct Invoice: Payable {
    let amount: Double
    // pas besoin de redéfinir describe() : hérité de l'extension du protocole
}

struct Refund: Payable {
    let amount: Double
}

print(Invoice(amount: 99.0).describe()) // "Montant : 99.0€"
print(Refund(amount: 20.0).describe())   // "Montant : 20.0€"
```

## 3.2 — Cycle de rétention et ARC

```swift
class Owner {
    var pet: Pet?
}
class Pet {
    var owner: Owner? // sans weak : cycle de rétention
}
```

Sans `weak`, `Owner` détient une référence forte vers `Pet` et `Pet` détient une référence forte vers `Owner` : le compteur de références (ARC) de chacun ne redescend jamais à zéro, même si plus aucune variable externe ne pointe vers eux — ils se maintiennent mutuellement en vie indéfiniment, une fuite mémoire silencieuse.

```swift
class Pet {
    weak var owner: Owner? // weak casse le cycle : ne compte pas comme référence forte
}
```

Le sens `Pet → Owner` (et pas l'inverse) est le bon choix car `Owner` est conceptuellement le "possesseur" de la relation (il crée/détient ses pets) : c'est la référence de l'enfant vers son parent qui doit être `weak`, pour que la suppression du parent libère naturellement l'enfant plutôt que l'inverse.

## 3.3 — Actor et concurrence

```swift
actor Counter {
    private var count = 0
    func increment() { count += 1 }
    func getValue() -> Int { count }
}

let counter = Counter()

await withTaskGroup(of: Void.self) { group in
    for _ in 1...100 {
        group.addTask { await counter.increment() }
    }
}

print(await counter.getValue()) // toujours 100, jamais de valeur perdue
```

Le compilateur garantit qu'un seul appel à la fois s'exécute à l'intérieur d'un `actor` (isolation automatique de son état interne) : deux appels concurrents à `increment()` sont sérialisés, éliminant la *data race* à la compilation. En Kotlin, une coroutine ne fournit aucune garantie équivalente par défaut sur un état mutable partagé — il faut explicitement confiner les accès à un `Dispatcher` unique ou utiliser un `Mutex`.

## 3.4 — Design orienté protocoles (comparaison avec Kotlin)

```swift
protocol UserRepository {
    func findById(_ id: String) -> User?
}

class MySQLUserRepository: UserRepository {
    func findById(_ id: String) -> User? {
        // requête SQL réelle
        return nil
    }
}

class InMemoryUserRepository: UserRepository {
    private var users: [String: User] = [:]
    func findById(_ id: String) -> User? { users[id] }
}

class UserService {
    private let repository: UserRepository // dépend du protocole, jamais de MySQLUserRepository directement
    init(repository: UserRepository) {
        self.repository = repository
    }
}
```

Syntaxiquement très proche de la version Kotlin (voir [`../../kotlin/solutions/niveau-3.md`](../../kotlin/solutions/niveau-3.md)) : `protocol` remplace `interface`, `: UserRepository` remplace `: UserRepository` (identique), et l'injection par constructeur suit le même schéma. La différence principale est l'absence de constructeur primaire concis en Swift : `init` doit être écrit explicitement, là où Kotlin permet de déclarer et affecter le champ directement dans l'en-tête de la classe.
