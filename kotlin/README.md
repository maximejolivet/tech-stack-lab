# Kotlin

## 1. Introduction

Kotlin est un langage compilé, orienté objet et fonctionnel, à typage statique fort, qui s'exécute sur la JVM et est **100% interopérable** avec Java — voir [`../java/`](../java/) pour les bases JVM communes (bytecode, packages, garbage collector). Ce dossier se concentre sur ce que Kotlin apporte **en plus ou différemment** de Java plutôt que de répéter les fondamentaux JVM déjà couverts.

**À quoi sert-il ?**
- Développement Android natif — langage officiellement recommandé par Google depuis 2019, a largement supplanté Java sur ce terrain.
- Backend JVM moderne, via des frameworks comme Ktor ou Spring Boot (support Kotlin de première classe).
- Kotlin Multiplatform (KMP) — partager de la logique métier entre Android, iOS, web et desktop depuis une base de code commune.

**Où se situe-t-il dans une architecture web ?**
Côté serveur, dans les mêmes conditions qu'une application Java : process JVM long-running, packagé en `.jar`, déployé sur un serveur ou conteneur. Rien ne change côté architecture par rapport à Java — la différence est uniquement dans l'expressivité du langage utilisé pour écrire la logique.

**Avantages**
- Null safety intégrée au système de types : la majorité des `NullPointerException` sont éliminées à la compilation plutôt que découvertes en production.
- Syntaxe nettement plus concise que Java pour les mêmes résultats (data classes, inférence de type, absence de point-virgules obligatoires).
- Interopérabilité totale avec l'écosystème Java existant (librairies, frameworks) — migration progressive possible fichier par fichier dans un même projet.

**Limites**
- Temps de compilation plus long que Java sur de gros projets (compilateur plus complexe).
- Écosystème de ressources d'apprentissage plus restreint que Java, bien que très actif.
- Certaines fonctionnalités avancées (coroutines, delegated properties) ont une courbe d'apprentissage réelle avant d'être utilisées correctement.

## 2. Prérequis

- Bases Java déjà vues ([`../java/`](../java/)) — JVM, compilation, POO, collections, généricité : Kotlin en hérite directement et ce dossier ne les re-explique pas.
- JDK installé en local (même prérequis que Java, `java -version`).
- Un IDE avec support Kotlin (IntelliJ IDEA, qui l'intègre nativement — logique, car Kotlin est développé par JetBrains, l'éditeur d'IntelliJ).

## 3. Rappel des bases 🟢

### 01 - Interopérabilité et compilation

**Explication** — Le code `.kt` est compilé par `kotlinc` en bytecode `.class`, strictement identique à celui produit par `javac` du point de vue de la JVM. Un fichier Kotlin peut appeler une classe Java et inversement, dans le même projet, sans configuration particulière.

```bash
kotlinc HelloWorld.kt -include-runtime -d HelloWorld.jar
java -jar HelloWorld.jar
```

**Cas d'usage** : migrer un projet Java vers Kotlin progressivement, classe par classe, sans réécriture complète — un cas d'usage réel très fréquent en entreprise.

### 02 - Structure minimale et fonction main

**Explication** — Contrairement à Java, une fonction n'a pas besoin d'être encapsulée dans une classe : `fun main()` au niveau fichier suffit comme point d'entrée.

```kotlin
fun main() {
    println("Hello, world!")
}
```

**Erreur fréquente** : chercher une classe publique obligatoire par réflexe Java — Kotlin compile ce fichier en générant une classe cachée (`HelloWorldKt`) automatiquement, invisible dans le code source.

### 03 - val vs var (immutabilité par défaut)

**Explication** — `val` déclare une référence en lecture seule (assignée une seule fois), `var` une référence réassignable. Contrairement à `final` en Java (optionnel, souvent oublié), l'idiome Kotlin pousse à utiliser `val` par défaut et `var` seulement quand la mutabilité est réellement nécessaire.

```kotlin
val name = "Max"       // référence immuable, type inféré en String
var age = 30
age = 31                // OK, var est réassignable
// name = "Autre"       // erreur de compilation : val ne peut pas être réassigné
```

**Bonne pratique** : toujours commencer par `val`, ne passer à `var` que si le compilateur force la réassignation — réduit mécaniquement les bugs d'état mutable partagé.

### 04 - Null safety

**Explication** — Le type `String` ne peut jamais contenir `null` ; seul `String?` (type nullable, suffixe `?`) le peut, et le compilateur force à gérer ce cas avant tout usage. C'est la différence la plus structurante avec Java.

```kotlin
var name: String = "Max"
// name = null              // erreur de COMPILATION : String n'accepte pas null

var nickname: String? = null // OK : type explicitement nullable

println(nickname?.length)    // safe call : renvoie null si nickname est null, pas de crash
println(nickname?.length ?: 0) // opérateur Elvis : valeur par défaut si null
println(nickname!!.length)   // "trust me" : force l'accès, lève NullPointerException si null (à éviter)
```

**Erreur fréquente** : abuser de `!!` pour faire taire le compilateur sans réfléchir au cas `null` — cela réintroduit exactement le risque de NPE que Kotlin est censé éliminer. Réserver `!!` aux cas où la nullité est structurellement impossible et prouvable.

### 05 - Types de base et inférence

**Explication** — Types similaires à Java (`Int`, `Double`, `Boolean`, `String`...) mais tous représentés comme des objets en surface (pas de distinction primitif/objet visible dans le code, le compilateur optimise en interne). L'inférence de type est beaucoup plus systématique qu'en Java.

```kotlin
val age = 30          // inféré Int
val price = 19.99     // inféré Double
val isActive = true    // inféré Boolean
val explicit: Long = 100L
```

**Cas d'usage** : typer explicitement uniquement quand l'inférence serait ambiguë ou quand le type doit être plus large que celui inféré (ex. `Long` au lieu d'`Int` pour un identifiant).

### 06 - Structures de contrôle (if et when comme expressions)

**Explication** — `if` et `when` (équivalent du `switch`) sont des **expressions** qui retournent une valeur, pas seulement des instructions — contrairement à Java où `if` ne retourne rien par défaut.

```kotlin
val age = 20
val status = if (age >= 18) "majeur" else "mineur"

val label = when (dayOfWeek) {
    6, 7 -> "Week-end"
    in 1..5 -> "Semaine"
    else -> "Invalide"
}
```

**Bonne pratique** : préférer `when`/`if` comme expression assignée directement plutôt que déclarer une variable `var` vide puis l'assigner dans chaque branche — plus concis et évite d'oublier une branche.

### 07 - Fonctions et arguments nommés

**Explication** — Les fonctions peuvent avoir des valeurs par défaut pour leurs paramètres et être appelées avec des arguments nommés dans n'importe quel ordre — Java ne propose ni l'un ni l'autre nativement.

```kotlin
fun greet(name: String, greeting: String = "Bonjour") = "$greeting, $name !"

greet("Max")                          // "Bonjour, Max !"
greet("Max", "Salut")                  // "Salut, Max !"
greet(greeting = "Hey", name = "Max")   // ordre libre grâce aux arguments nommés
```

**Cas d'usage** : les paramètres par défaut évitent l'explosion de surcharges (`greet(String)`, `greet(String, String)`...) qu'impose Java pour le même besoin.

### 08 - Chaînes de caractères et templates

**Explication** — Les *string templates* insèrent directement des expressions dans une chaîne avec `$variable` ou `${expression}`, sans concaténation manuelle.

```kotlin
val name = "Max"
val age = 30
println("Bonjour $name, tu as ${age + 1} ans l'année prochaine.")

val html = """
    <html>
        <body>Hello</body>
    </html>
""".trimIndent() // raw string multi-ligne, trimIndent() retire l'indentation de code
```

**Erreur fréquente** : oublier `trimIndent()` sur une raw string multi-ligne — le texte conserve alors l'indentation du code source, rarement le résultat voulu.

### 09 - Collections de base

**Explication** — Distinction explicite entre collections **immuables** (`List`, `Map`, `Set` — lecture seule par défaut) et **mutables** (`MutableList`, `MutableMap`, `MutableSet`), contrairement à Java où `List` est mutable par défaut sauf `List.of()`.

```kotlin
val fixedList = listOf("Alice", "Bob")       // immuable : pas de .add() possible
val editableList = mutableListOf("Alice")
editableList.add("Bob")                        // OK

val ages = mapOf("Alice" to 30, "Bob" to 25)  // Map immuable, syntaxe "to" pour les paires
println(ages["Bob"])                            // 25
println(ages["Charlie"] ?: 0)                    // valeur par défaut si clé absente
```

## 4. Concepts intermédiaires 🟡

- **Classes et data classes** : une `class` classique fonctionne comme en Java (constructeur primaire dans l'en-tête). Une `data class` génère automatiquement `equals()`/`hashCode()`/`toString()`/`copy()` — équivalent quasi direct des `record` Java, mais disponible depuis bien plus longtemps et avec mutabilité possible (`var` dans le constructeur).

```kotlin
data class User(val name: String, val age: Int)

val user1 = User("Max", 30)
val user2 = user1.copy(age = 31) // nouvelle instance, seul age change
println(user1) // User(name=Max, age=30) — toString() généré automatiquement
```

- **Extension functions** : ajouter une méthode à une classe existante (même une classe de librairie tierce ou du JDK) sans hériter ni la modifier — impossible en Java sans wrapper ou classe utilitaire statique.

```kotlin
fun String.isPalindrome(): Boolean = this == this.reversed()

println("radar".isPalindrome()) // true — appelée comme une méthode native de String
```

- **Objects et companion object** : `object` déclare un singleton directement (pas de pattern Singleton manuel à écrire). `companion object` à l'intérieur d'une classe fournit l'équivalent des membres `static` de Java, mais reste un vrai objet (peut implémenter une interface).

```kotlin
object AppConfig {
    val version = "1.0"
}
println(AppConfig.version) // singleton, une seule instance existe

class User private constructor(val name: String) {
    companion object {
        fun create(name: String) = User(name) // équivalent d'une factory static Java
    }
}
```

- **Sealed classes et when exhaustif** : comme les *sealed classes* Java 17+, mais présentes depuis les débuts de Kotlin. Combinées à un `when` sans `else`, le compilateur garantit que tous les cas sont couverts.

```kotlin
sealed class PaymentEvent
data class Succeeded(val orderId: String, val amount: Double) : PaymentEvent()
data class Failed(val orderId: String, val reason: String) : PaymentEvent()

fun describe(event: PaymentEvent): String = when (event) {
    is Succeeded -> "Commande ${event.orderId} payée : ${event.amount}€"
    is Failed -> "Commande ${event.orderId} échouée : ${event.reason}"
    // pas de "else" nécessaire : le compilateur refuse de compiler si un cas manque
}
```

- **Fonctions d'ordre supérieur, lambdas et scope functions** : les fonctions sont des citoyens de première classe (paramètres, valeurs de retour). Les *scope functions* (`let`, `run`, `apply`, `also`, `with`) structurent des blocs de code autour d'un objet — idiome très fréquent en Kotlin, sans équivalent direct en Java.

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val doubled = numbers.map { it * 2 }.filter { it > 4 }

val user = User("Max", 30).apply {
    println("Création de $name") // "apply" retourne l'objet lui-même, utile pour la config/init
}

nickname?.let { println("Pseudo : $it") } // exécute le bloc seulement si nickname n'est pas null
```

- **Interopérabilité Java-Kotlin** : appeler du code Java depuis Kotlin (et inversement) est transparent, avec quelques nuances — une méthode Java retournant un type sans annotation de nullabilité est traitée comme un "type plateforme" (`String!`), où le compilateur Kotlin ne peut pas garantir l'absence de `null` : la vigilance manuelle reste nécessaire à cette frontière.

## 5. Concepts avancés 🟠🔴

- **Coroutines** : mécanisme de concurrence léger basé sur des fonctions `suspend`, permettant d'écrire du code asynchrone de façon séquentielle (sans callbacks imbriqués). Contrairement aux `Thread` Java (lourds, un par tâche), des milliers de coroutines peuvent tourner sur un pool de quelques threads réels.

```kotlin
suspend fun fetchUser(id: String): User {
    delay(500) // suspend l'exécution sans bloquer le thread, contrairement à Thread.sleep
    return User("Max", 30)
}

fun main() = runBlocking {
    val user = async { fetchUser("1") } // lancée en parallèle
    val posts = async { fetchPosts("1") }
    println("${user.await()} - ${posts.await()}") // attend les deux résultats
}
```

- **Flow** : flux de données asynchrones émis dans le temps (équivalent conceptuel d'un Observable RxJava ou d'un stream réactif), construit sur les coroutines — utile pour représenter une séquence de valeurs qui arrivent progressivement (résultats de recherche, mises à jour temps réel).
- **Delegated properties** : déléguer la logique d'accès à une propriété via `by` — `lazy` (calcul différé au premier accès), `Delegates.observable` (callback à chaque changement) sont fournis par la stdlib, et un délégué custom est possible.

```kotlin
val expensiveValue: String by lazy {
    println("Calcul effectué une seule fois")
    computeExpensiveValue()
}
```

- **Inline functions et reified generics** : `inline` copie le corps d'une fonction lambda directement au site d'appel (élimine le coût d'allocation d'objet lambda), et permet le mot-clé `reified` pour accéder au type générique à l'exécution — impossible en Java à cause de l'effacement de type (*type erasure*).

```kotlin
inline fun <reified T> isInstance(value: Any): Boolean = value is T
// reified T : le type générique reste connu à l'exécution grâce à l'inlining
```

- **Kotlin Multiplatform (KMP)** : partager de la logique métier (validation, modèles, appels réseau) entre Android, iOS, backend et web depuis une base de code Kotlin commune, chaque plateforme gardant son UI native — approche différente du "write once" React Native/Flutter, ici seule la logique est partagée, pas l'UI.
- **Écosystème serveur** : Ktor (framework HTTP léger, développé par JetBrains, basé sur les coroutines) et le support Kotlin de première classe dans Spring Boot ([`../springboot/`](../springboot/)) — les deux options dominantes pour un backend Kotlin en production.

## 6. Commandes / syntaxe à connaître

```bash
kotlinc -version                        # version du compilateur

kotlinc HelloWorld.kt -include-runtime -d app.jar  # compiler en jar exécutable
java -jar app.jar                        # exécuter

kotlinc-jvm -script script.kts           # exécuter un script Kotlin directement (.kts)

# Gradle (build system dominant pour Kotlin, DSL Kotlin possible : build.gradle.kts)
gradle build                              # build complet
gradle test                               # lancer les tests
gradle run                                # exécuter l'application
```

```kotlin
// Syntaxe essentielle à avoir sous les doigts
val list = listOf(1, 2, 3).map { it * 2 }.filter { it > 2 }
data class Point(val x: Int, val y: Int)
val result = nickname?.let { it.uppercase() } ?: "ANONYME"
when (value) {
    is String -> println("texte")
    is Int -> println("nombre")
    else -> println("autre")
}
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Gestionnaire de bibliothèque en CLI, orienté objet**

Reprendre le mini-projet Java ([`../java/README.md`](../java/README.md)) en Kotlin idiomatique, pour comparer directement les deux langages sur le même exercice :
- Modéliser un livre avec une `data class Book(val isbn: String, val title: String, val available: Boolean)`.
- Stocker les livres dans une `MutableList<Book>` en mémoire, avec des opérations via les fonctions de collection (`filter`, `find`).
- Utiliser une hiérarchie `sealed class LibraryAction` (`BorrowAction`, `ReturnAction`) traitée avec un `when` exhaustif.
- Gérer une absence de résultat avec `Book?` et un safe call plutôt qu'une exception, là où c'est pertinent.
- Lire les commandes depuis `readLine()` (boucle interactive simple).

Objectif : mobiliser data classes, sealed classes/when exhaustif, null safety et fonctions de collection dans le même exercice concret que la version Java, pour ressentir concrètement le gain de concision et de sécurité de Kotlin.

## Checklist

- [ ] Comprendre les fondamentaux (interopérabilité JVM, val/var, null safety)
- [ ] Savoir créer un projet Kotlin avec Gradle
- [ ] Maîtriser la syntaxe principale (data classes, when, extension functions)
- [ ] Comprendre les concepts importants (sealed classes, scope functions, interop Java)
- [ ] Savoir debugger (breakpoints IDE, stack traces JVM)
- [ ] Connaître les bonnes pratiques (val par défaut, éviter `!!`, safe calls)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (coroutines, Flow, delegated properties, KMP)

## 10. Ressources

- [Documentation officielle Kotlin](https://kotlinlang.org/docs/home.html) — référence complète du langage.
- [Kotlin Koans](https://play.kotlinlang.org/koans) — exercices interactifs officiels pour pratiquer la syntaxe.
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html) — guide officiel sur les coroutines et Flow.
- [Ktor Documentation](https://ktor.io/docs/) — framework HTTP Kotlin léger pour le backend.
- [roadmap.sh — Kotlin](https://roadmap.sh/kotlin) — vue d'ensemble du parcours d'apprentissage.
