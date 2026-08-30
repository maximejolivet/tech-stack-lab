# Niveau 1 — Bases

## Exercice 1.1 — Hello, typé

Écris une fonction `main` qui déclare un `val name: String`, un `val age: Int`, et affiche `"Bonjour <name>, tu as <age> ans."` en utilisant un string template (pas de concaténation `+`).

## Exercice 1.2 — Null safety

Sans exécuter le code, prédis le résultat de chaque ligne, puis vérifie :

```kotlin
val nickname: String? = null

println(nickname?.length)
println(nickname?.length ?: -1)
println(nickname?.uppercase()?.length ?: 0)
```

Explique en une phrase la différence entre `?.` et `?:`.

## Exercice 1.3 — Collections immuables vs mutables

Déclare une `val scores = listOf(12, 15, 8, 18, 10)`. Tente d'ajouter un élément avec `.add()` et observe l'erreur de compilation. Corrige en utilisant le bon type de collection, puis calcule la moyenne avec `.average()`.

## Exercice 1.4 — When expression

Écris une fonction `fun dayType(day: Int): String` qui retourne `"Week-end"` pour 6 et 7, `"Semaine"` pour 1 à 5, `"Invalide"` sinon, en utilisant `when` comme **expression** (valeur retournée directement, pas de `return` dans chaque branche).

## Exercice 1.5 — Fonction avec argument par défaut

Écris une fonction `fun greet(name: String, greeting: String = "Bonjour"): String` et appelle-la trois fois : sans deuxième argument, avec un deuxième argument positionnel, puis avec des arguments nommés dans l'ordre inverse.
