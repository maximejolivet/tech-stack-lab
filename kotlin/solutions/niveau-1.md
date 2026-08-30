# Solutions — Niveau 1

## 1.1 — Hello, typé

```kotlin
fun main() {
    val name = "Max"
    val age = 30
    println("Bonjour $name, tu as $age ans.")
}
```

## 1.2 — Null safety

```kotlin
println(nickname?.length)                        // null (safe call sur null → null, pas de crash)
println(nickname?.length ?: -1)                   // -1 (Elvis fournit la valeur par défaut)
println(nickname?.uppercase()?.length ?: 0)        // 0 (chaîne de safe calls, s'arrête au premier null)
```

`?.` (safe call) renvoie `null` immédiatement si l'objet est `null` au lieu de lever une exception ; `?:` (opérateur Elvis) fournit une valeur de remplacement quand l'expression à sa gauche vaut `null`.

## 1.3 — Collections immuables vs mutables

```kotlin
val scores = listOf(12, 15, 8, 18, 10)
// scores.add(20) // erreur de COMPILATION : List n'expose pas .add()

val editableScores = mutableListOf(12, 15, 8, 18, 10)
editableScores.add(20) // OK

println(scores.average()) // 12.6
```

## 1.4 — When expression

```kotlin
fun dayType(day: Int): String = when (day) {
    6, 7 -> "Week-end"
    in 1..5 -> "Semaine"
    else -> "Invalide"
}
```

## 1.5 — Fonction avec argument par défaut

```kotlin
fun greet(name: String, greeting: String = "Bonjour"): String = "$greeting, $name !"

println(greet("Max"))                          // "Bonjour, Max !"
println(greet("Max", "Salut"))                  // "Salut, Max !"
println(greet(greeting = "Hey", name = "Max"))   // "Hey, Max !" — ordre libre grâce aux noms
```
