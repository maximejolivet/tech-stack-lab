# Niveau 2 — Intermédiaire

## Exercice 2.1 — Data class

Crée une `data class Product(val name: String, val price: Double)`. Crée une instance, puis une copie avec un prix différent via `.copy()`. Affiche les deux instances avec `println` (sans définir `toString()` toi-même) et vérifie qu'elles sont différentes avec `==`.

## Exercice 2.2 — Extension function

Écris une extension function `fun Int.isEven(): Boolean` qui retourne `true` si le nombre est pair. Utilise-la pour filtrer une `List<Int>` et ne garder que les nombres pairs (avec `.filter`).

## Exercice 2.3 — Sealed class et when exhaustif

Modélise `sealed class Shape` avec deux sous-types `data class Circle(val radius: Double)` et `data class Rectangle(val width: Double, val height: Double)`. Écris une fonction `fun area(shape: Shape): Double` avec un `when` **exhaustif** (sans `else`) qui calcule l'aire selon le type réel.

## Exercice 2.4 — Scope functions

Utilise `apply` pour construire et initialiser un objet `Product` (paramètre `var` sur au moins un champ) en une seule expression chaînée. Utilise ensuite `let` pour n'afficher un message que si une variable nullable `discount: Double?` n'est pas `null`.

## Exercice 2.5 — Companion object

Crée une classe `User` avec un constructeur `private`, et un `companion object` exposant une fonction `fun create(name: String): User` qui construit l'instance (pattern factory). Vérifie qu'il est impossible d'appeler `User(...)` directement depuis l'extérieur de la classe.
