# Niveau 3 — Avancé

## Exercice 3.1 — Coroutines et async/await

Écris deux fonctions `suspend fun fetchUser(): String` et `suspend fun fetchPosts(): String`, chacune simulant une latence avec `delay(500)`. Depuis `fun main() = runBlocking { ... }`, lance les deux en parallèle avec `async`, récupère les résultats avec `.await()`, et affiche le temps total écoulé. Explique en une phrase pourquoi ce temps est proche de 500ms et pas de 1000ms.

## Exercice 3.2 — Delegated property avec lazy

Écris une propriété `val config: Map<String, String> by lazy { ... }` dont le bloc d'initialisation affiche `"Chargement de la config"` avant de retourner une map. Accède à `config` deux fois de suite et observe que le message ne s'affiche qu'une seule fois.

## Exercice 3.3 — Reified generics

Écris une fonction `inline fun <reified T> Any.isInstance(): Boolean` qui retourne `true` si l'objet appelant est une instance de `T`. Teste-la avec plusieurs types (`String`, `Int`, une data class custom). Explique en 2-3 lignes pourquoi ceci serait impossible en Java à cause du *type erasure*, en t'appuyant sur [`../../java/`](../../java/) (section génériques).

## Exercice 3.4 — Design orienté interfaces (comparaison avec Java)

Reprends l'exercice 3.4 du dossier Java ([`../../java/exercices/niveau-3.md`](../../java/exercices/niveau-3.md)) : une classe `UserService` qui dépend directement de `MySQLUserRepository`. Refactore-la en Kotlin pour dépendre d'une `interface UserRepository`, avec `MySQLUserRepository` et `InMemoryUserRepository` qui l'implémentent toutes les deux. Compare en 2-3 lignes la verbosité de ta version Kotlin avec l'équivalent Java.
