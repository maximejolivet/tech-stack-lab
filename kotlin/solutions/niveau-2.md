# Solutions — Niveau 2

## 2.1 — Data class

```kotlin
data class Product(val name: String, val price: Double)

val product1 = Product("Clavier", 49.99)
val product2 = product1.copy(price = 39.99) // nouvelle instance, seul price change

println(product1) // Product(name=Clavier, price=49.99) — toString() généré
println(product2) // Product(name=Clavier, price=39.99)
println(product1 == product2) // false — equals() généré compare les champs, ici price diffère
```

## 2.2 — Extension function

```kotlin
fun Int.isEven(): Boolean = this % 2 == 0

val numbers = listOf(1, 2, 3, 4, 5, 6)
val evens = numbers.filter { it.isEven() } // [2, 4, 6]
```

## 2.3 — Sealed class et when exhaustif

```kotlin
sealed class Shape
data class Circle(val radius: Double) : Shape()
data class Rectangle(val width: Double, val height: Double) : Shape()

fun area(shape: Shape): Double = when (shape) {
    is Circle -> Math.PI * shape.radius * shape.radius
    is Rectangle -> shape.width * shape.height
    // pas de "else" : le compilateur refuse de compiler si un sous-type de Shape manque
}
```

## 2.4 — Scope functions

```kotlin
data class Product(val name: String, var discount: Double? = null)

val product = Product("Clavier").apply {
    discount = 10.0 // "apply" retourne l'objet lui-même après ce bloc de configuration
}

val discount: Double? = product.discount
discount?.let { println("Réduction appliquée : $it%") } // affiché seulement si non null
```

## 2.5 — Companion object

```kotlin
class User private constructor(val name: String) {
    companion object {
        fun create(name: String) = User(name)
    }
}

val user = User.create("Max") // seule façon d'obtenir une instance
// val other = User("Max")   // erreur de compilation : constructeur privé, inaccessible de l'extérieur
```
