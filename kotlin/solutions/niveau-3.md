# Solutions — Niveau 3

## 3.1 — Coroutines et async/await

```kotlin
suspend fun fetchUser(): String {
    delay(500)
    return "Max"
}

suspend fun fetchPosts(): String {
    delay(500)
    return "3 posts"
}

fun main() = runBlocking {
    val start = System.currentTimeMillis()

    val user = async { fetchUser() }   // démarre immédiatement, sans bloquer
    val posts = async { fetchPosts() }  // démarre en parallèle du précédent

    println("${user.await()} - ${posts.await()}") // attend les deux résultats

    println("Durée : ${System.currentTimeMillis() - start}ms")
}
```

Les deux appels `delay(500)` s'exécutent en parallèle (chacun dans sa propre coroutine, lancée par `async` avant que `.await()` ne soit appelé) : le temps total est donc proche de la durée du plus long des deux (~500ms), pas de leur somme (1000ms) comme ce serait le cas en séquentiel.

## 3.2 — Delegated property avec lazy

```kotlin
val config: Map<String, String> by lazy {
    println("Chargement de la config")
    mapOf("env" to "production")
}

fun main() {
    println(config) // affiche "Chargement de la config" PUIS la map
    println(config) // affiche seulement la map : le bloc lazy ne s'exécute qu'une fois, résultat mis en cache
}
```

## 3.3 — Reified generics

```kotlin
data class Point(val x: Int, val y: Int)

inline fun <reified T> Any.isInstance(): Boolean = this is T

println("hello".isInstance<String>()) // true
println(42.isInstance<String>())       // false
println(Point(1, 2).isInstance<Point>()) // true
```

En Java, les génériques sont effacés à la compilation (*type erasure*) : à l'exécution, une `List<String>` et une `List<Integer>` sont toutes deux simplement une `List`, le type `T` n'existe plus et un test `value instanceof T` est interdit par le compilateur. En Kotlin, `inline` copie le corps de la fonction à chaque site d'appel avec le type concret substitué, donc `reified T` reste connu à l'exécution — un contournement au niveau du compilateur, pas une suppression du type erasure de la JVM elle-même.

## 3.4 — Design orienté interfaces (comparaison avec Java)

```kotlin
interface UserRepository {
    fun findById(id: String): User?
}

class MySQLUserRepository : UserRepository {
    override fun findById(id: String): User? {
        // requête SQL réelle
        return null
    }
}

class InMemoryUserRepository : UserRepository {
    private val users = mutableMapOf<String, User>()
    override fun findById(id: String): User? = users[id]
}

class UserService(private val repository: UserRepository) {
    // dépend de l'interface, jamais de MySQLUserRepository directement
}
```

Même principe de Dependency Inversion qu'en Java (voir [`../../java/solutions/niveau-3.md`](../../java/solutions/niveau-3.md)), mais sensiblement plus court : pas de champ `final` explicite ni de constructeur écrit à la main (le constructeur primaire de la classe fait les deux en une ligne), et `findById` retourne directement `User?` (null safety) là où Java doit passer par `Optional<User>` pour exprimer la même intention.
