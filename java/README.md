# Java

## 1. Introduction

Java est un langage compilé, orienté objet, à typage statique fort, qui s'exécute sur la JVM (Java Virtual Machine) plutôt que directement sur le système d'exploitation. Ce dossier couvre le **langage et la JVM** — le framework [`../springboot/`](../springboot/) (l'équivalent Symfony/Laravel côté Java) a son propre dossier juste après.

**À quoi sert-il ?**
- Applications backend d'entreprise (API, systèmes bancaires, e-commerce à grande échelle) — c'est le langage dominant du "Java shop" corporate.
- Applications Android (Kotlin l'a largement supplanté depuis, mais l'écosystème Java reste sous-jacent).
- Systèmes distribués et big data (Hadoop, Kafka, Spark sont écrits en JVM).

**Où se situe-t-il dans une architecture web ?**
Côté serveur exclusivement, comme PHP : il reçoit une requête, exécute de la logique métier, retourne une réponse. Contrairement à PHP-FPM (process tué à chaque requête), une application Java tourne en continu dans un process JVM long-running — plus proche du modèle Node.js dans ce sens précis.

**Avantages**
- Typage statique fort : la majorité des erreurs de type sont détectées à la compilation, pas au runtime (différence structurante avec PHP/JS).
- Écosystème mature et stable, rétrocompatibilité prise très au sérieux (du code Java 8 tourne souvent tel quel sur Java 21).
- Performance et gestion mémoire prévisible pour du code long-running, grâce à un garbage collector très optimisé après 25 ans d'itérations.

**Limites**
- Verbosité historique (boilerplate des getters/setters, `public static void main`) — largement réduite par les versions récentes (records, var, text blocks).
- Cycle compile → package → déployer plus lourd que PHP (pas de simple rechargement de fichier en prod).
- Startup JVM plus lent qu'un runtime interprété classique (impact réel sur du serverless/CLI, moins sur un serveur long-running).

## 2. Prérequis

- POO déjà pratiquée en PHP ([`../php/`](../php/), section 4) — les concepts (classes, interfaces, héritage) sont transférables, la syntaxe et la rigueur du typage changent.
- JDK installé en local (`java -version`, `javac -version`) — JDK 21 (LTS) recommandé.
- Un IDE avec support Java (IntelliJ IDEA recommandé, gratuit en édition Community) plutôt qu'un simple éditeur de texte : l'autocomplétion et la détection d'erreurs sont beaucoup plus utiles ici qu'en PHP dynamique.

## 3. Rappel des bases 🟢

### 01 - La JVM et le principe "write once, run anywhere"

**Explication** — Le code source `.java` est compilé par `javac` en bytecode `.class`, exécuté par la JVM (`java`). La JVM est spécifique à chaque OS/architecture, mais le bytecode est universel : le même `.class` tourne sur Windows, Linux ou macOS sans recompilation.

```bash
javac HelloWorld.java   # compile → génère HelloWorld.class (bytecode)
java HelloWorld           # exécute le bytecode via la JVM
```

**Cas d'usage** : comprendre cette étape explique pourquoi Java nécessite une étape de build (contrairement à PHP interprété directement) et pourquoi un `.jar`/`.war` packagé contient du bytecode, pas du code source.

**Bonne pratique** : ne jamais versionner les fichiers `.class` générés (comme `vendor/` en PHP) — seul le code source et la config de build (Maven/Gradle) doivent être commités.

### 02 - Structure minimale et méthode main

**Explication** — Un fichier `.java` doit porter le même nom que sa classe publique. Le point d'entrée d'un programme est la méthode `main`, avec une signature fixe.

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

**Erreur fréquente** : nommer le fichier différemment de la classe publique qu'il contient (`Foo.java` contenant `public class Bar`) — erreur de compilation immédiate.

**Bonne pratique** : une classe publique par fichier ; les classes utilitaires internes non publiques peuvent cohabiter dans le même fichier si elles sont étroitement liées.

### 03 - Types primitifs vs objets (boxing/unboxing)

**Explication** — Java distingue les **types primitifs** (`int`, `double`, `boolean`, `char`, `long`, `float`, `byte`, `short`) — stockés directement, performants — des **objets** (`Integer`, `Double`, `Boolean`...) — wrappers qui permettent d'utiliser ces valeurs comme objets (dans une `List`, par exemple, qui ne peut contenir que des objets).

```java
int a = 5;                 // primitif
Integer b = 5;              // objet (auto-boxing implicite)
Integer c = Integer.valueOf(5);

List<Integer> nums = new ArrayList<>(); // List<int> est impossible : il faut le wrapper
nums.add(5); // auto-boxing : 5 (int) → Integer
```

**Erreur fréquente** : comparer deux `Integer` avec `==` en pensant comparer des valeurs — `==` compare des références d'objet, pas des valeurs, en dehors du cache interne `[-128, 127]` de la JVM.

```java
Integer x = 200, y = 200;
System.out.println(x == y);      // false ! (objets différents hors du cache)
System.out.println(x.equals(y)); // true (comparaison de valeur)
```

**Bonne pratique** : toujours utiliser `.equals()` pour comparer des objets (y compris les wrappers), réserver `==` aux types primitifs ou aux comparaisons de référence volontaires.

### 04 - Déclaration de variables et typage statique

**Explication** — Chaque variable a un type déclaré, fixe à vie (contrairement à `let`/`$var`). Depuis Java 10, `var` permet l'inférence de type locale, mais le type reste statique et déterminé à la compilation — ce n'est PAS un équivalent du `var` JS dynamique.

```java
int age = 30;
String name = "Max";
var city = "Paris";  // inféré en String à la compilation, reste String pour toujours

// age = "trente"; // erreur de COMPILATION, pas de runtime : impossible d'assigner un autre type
```

**Cas d'usage** : `var` améliore la lisibilité quand le type est déjà évident du contexte (`var list = new ArrayList<String>();`), à éviter quand il obscurcit le type retourné par une méthode peu explicite.

**Bonne pratique** : typer explicitement les signatures publiques (paramètres, retours) même avec `var` disponible pour les variables locales — la clarté de l'API prime.

### 05 - Structures de contrôle

**Explication** — `if/else`, `switch` (classique et l'expression `switch` moderne depuis Java 14), `for`, `while`, `do...while` — syntaxe très proche de PHP/JS avec accolades obligatoires en style idiomatique.

```java
int age = 20;
String status = switch (true) {
    case true when age >= 18 -> "majeur";
    default -> "mineur";
};

// forme la plus courante : switch sur une valeur
String label = switch (dayOfWeek) {
    case 1, 7 -> "Week-end";
    default -> "Semaine";
};
```

**Erreur fréquente** : utiliser encore le vieux `switch` avec `case: ... break;` par habitude alors que la forme flèche (`->`) moderne élimine le risque de fall-through oublié — comparable au `match` de PHP 8 face au vieux `switch`.

### 06 - Tableaux

**Explication** — Taille fixe à la création, type unique, syntaxe `Type[]`. Pour une collection dynamique, voir `ArrayList` en intermédiaire.

```java
int[] scores = {90, 85, 77};
String[] names = new String[3]; // tableau de 3 éléments, initialisés à null
names[0] = "Alice";

System.out.println(scores.length); // pas de parenthèses : .length est un champ, pas une méthode
```

**Erreur fréquente** : appeler `.length()` (avec parenthèses, comme sur une `String`) sur un tableau — `.length` sur un tableau est un champ, `.length()` est une méthode réservée à `String`. Confusion source d'erreur de compilation fréquente pour un débutant.

### 07 - Chaînes de caractères

**Explication** — `String` est immuable (comme en PHP) : toute "modification" crée une nouvelle instance. `StringBuilder` pour les concaténations répétées en boucle (évite de créer N objets intermédiaires).

```java
String greeting = "Bonjour, " + "Max" + " !";

StringBuilder sb = new StringBuilder();
for (String word : List.of("un", "deux", "trois")) {
    sb.append(word).append(" ");
}
String result = sb.toString();

// text block (Java 15+), pour du texte multi-ligne
String html = """
    <html>
        <body>Hello</body>
    </html>
    """;
```

**Erreur fréquente** : concaténer des chaînes avec `+` dans une boucle de grande taille — chaque `+` crée un nouvel objet `String` immuable, coût O(n²) sur la boucle entière. Utiliser `StringBuilder` dès que la boucle dépasse quelques itérations.

### 08 - Packages et imports

**Explication** — Un `package` regroupe des classes liées, mappé sur l'arborescence de dossiers (équivalent conceptuel des namespaces PSR-4 en PHP). `import` rend une classe d'un autre package utilisable sans son nom complet.

```java
package com.example.app.model;

import java.util.List;
import java.util.ArrayList;

public class User {
    // ...
}
```

**Bonne pratique** : convention de nommage inversée du domaine (`com.example.app`), un package par couche/domaine logique — cohérent avec l'usage des namespaces `App\Domain\...` en PHP moderne.

## 4. Concepts intermédiaires 🟡

- **POO en profondeur** : `class`, `interface` (contrat, une classe peut en implémenter plusieurs — comme PHP), `abstract class` (héritage simple uniquement), héritage `extends`, polymorphisme (une méthode surchargée `@Override` se comporte selon le type réel de l'objet à l'exécution), encapsulation (`private`/`protected`/`public`, pas de mot-clé équivalent au `readonly` PHP avant les **records**, voir avancé).

```java
public interface PaymentMethod {
    boolean pay(double amount);
}

public class CreditCard implements PaymentMethod {
    @Override
    public boolean pay(double amount) {
        // logique de paiement
        return true;
    }
}
```

- **Collections** : `List` (ordonnée, doublons permis — `ArrayList` accès rapide par index, `LinkedList` insertions fréquentes), `Set` (pas de doublons — `HashSet` non ordonné, `LinkedHashSet` ordre d'insertion), `Map` (clé-valeur — `HashMap`, équivalent d'un tableau associatif PHP).

```java
List<String> names = new ArrayList<>(List.of("Alice", "Bob"));
Map<String, Integer> ages = new HashMap<>();
ages.put("Alice", 30);
ages.getOrDefault("Bob", 0); // équivalent du ?? de PHP/JS
```

- **Génériques** : `List<T>` plutôt qu'un `List` brut — le compilateur garantit le type contenu, élimine une classe entière de `ClassCastException` runtime que PHP ne peut pas prévenir statiquement.
- **Exceptions checked vs unchecked** : différence structurante avec PHP — une exception `checked` (hérite de `Exception` mais pas de `RuntimeException`) DOIT être déclarée (`throws`) ou catchée, sinon erreur de compilation ; une `unchecked` (`RuntimeException` et ses sous-classes) se comporte comme en PHP (propagation libre).

```java
public void readFile(String path) throws IOException { // obligatoire : IOException est checked
    Files.readString(Path.of(path));
}

try {
    readFile("data.txt");
} catch (IOException e) {
    // le compilateur FORCE ce traitement (ou la propagation via throws)
}
```

- **Streams API et lambdas** : programmation fonctionnelle sur les collections, très proche des `array_map`/`array_filter`/`array_reduce` PHP ou `.map`/`.filter`/`.reduce` JS.

```java
List<Integer> prices = List.of(10, 25, 8, 42);

List<Integer> withTax = prices.stream()
    .map(p -> (int) (p * 1.2))
    .filter(p -> p > 20)
    .toList(); // Java 16+
```

- **Build avec Maven ou Gradle** : `pom.xml` (Maven, déclaratif XML) ou `build.gradle` (Gradle, DSL Groovy/Kotlin) déclarent dépendances, plugins, et étapes de build — équivalent de `composer.json` mais avec un cycle de vie de build plus riche (compile, test, package).

## 5. Concepts avancés 🟠🔴

- **Records** (Java 16+) : classes immuables de données, génèrent automatiquement constructeur, getters, `equals`/`hashCode`/`toString` — équivalent quasi direct des propriétés `readonly` + constructor promotion de PHP 8.

```java
public record Point(int x, int y) {}

Point p = new Point(3, 4);
p.x(); // 3 — pas de "getX()", juste le nom du champ en méthode
```

- **Sealed classes** (Java 17+) : restreignent explicitement quelles classes peuvent hériter/implémenter — combiné à un `switch` exhaustif, le compilateur garantit que tous les cas sont couverts (proche de l'exhaustivité du `match` PHP sur un `enum`).
- **Garbage collector et mémoire** : la JVM gère un *heap* (objets) divisé en générations (young/old), le GC nettoie automatiquement les objets inaccessibles. Contrairement à PHP (process tué à chaque requête, mémoire repartant de zéro), une app Java accumule un état mémoire sur toute sa durée de vie : les fuites mémoire (references non relâchées, listeners jamais désenregistrés) ont un impact cumulatif bien plus grave.
- **Concurrence** : `Thread` (bas niveau), `ExecutorService` (pool de threads géré), `synchronized` (verrou d'exclusion mutuelle sur un bloc/méthode) — modèle à threads réels multi-cœurs, fondamentalement différent de l'event loop mono-thread de Node.js : Java peut exécuter du code en parallèle CPU-bound nativement, Node doit déléguer à des Worker Threads séparés pour la même chose.

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> processTask(task));
executor.shutdown();
```

- **Performance et JIT** : le bytecode est d'abord interprété, puis le JIT (Just-In-Time compiler) compile à la volée en code machine natif le code "chaud" (exécuté fréquemment) — explique pourquoi une app Java devient plus rapide après quelques minutes d'exécution ("warm-up"), contrairement à PHP où chaque requête repart de zéro.
- **Écosystème JVM** : Kotlin (langage moderne interopérable à 100% avec Java, dominant sur Android aujourd'hui), Scala, Clojure — tous compilent vers le même bytecode et peuvent s'appeler mutuellement, un atout unique de la JVM.

## 6. Commandes / syntaxe à connaître

```bash
java -version                 # version du runtime JVM
javac -version                 # version du compilateur

javac HelloWorld.java          # compiler un fichier
java HelloWorld                 # exécuter (sans extension .class)
java HelloWorld.java             # Java 11+ : compile ET exécute en une commande (scripts/prototypage)

# Maven
mvn compile                     # compiler le projet
mvn test                        # lancer les tests
mvn package                     # générer le .jar/.war
mvn clean install               # nettoyer + compiler + tester + installer en repo local

# Gradle
gradle build                    # build complet (compile + test + package)
gradle test                     # lancer les tests
```

```java
// Syntaxe essentielle à avoir sous les doigts
var list = new ArrayList<String>();
list.stream().filter(s -> !s.isEmpty()).toList();
Optional<User> user = repository.findById(id); // pas de null direct sur un retour "peut être absent"
record Point(int x, int y) {}
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Gestionnaire de bibliothèque en CLI, orienté objet**

Construire une petite application Java en ligne de commande (sans framework) qui doit :
- Modéliser un livre avec un `record Book(String isbn, String title, boolean available)`.
- Stocker les livres dans une `List<Book>` en mémoire, avec des opérations via `Stream` (recherche par titre, filtrage des disponibles).
- Utiliser une `interface LibraryAction` implémentée par plusieurs classes (`BorrowAction`, `ReturnAction`) pour illustrer le polymorphisme.
- Gérer une exception métier custom `BookNotFoundException` (checked), déclarée et catchée proprement.
- Lire les commandes depuis `Scanner` sur `System.in` (boucle interactive simple).

Objectif : mobiliser records, interfaces/polymorphisme, Streams, exceptions checked et collections dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux (JVM, types primitifs vs objets, structures de contrôle)
- [ ] Savoir créer un projet Java avec Maven ou Gradle
- [ ] Maîtriser la syntaxe principale (POO, collections, exceptions)
- [ ] Comprendre les concepts importants (génériques, Streams, checked vs unchecked)
- [ ] Savoir debugger (breakpoints IDE, stack traces)
- [ ] Connaître les bonnes pratiques (equals() vs ==, StringBuilder, immutabilité)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (records, sealed classes, GC, concurrence)

## 10. Ressources

- [Documentation officielle Java SE (Oracle)](https://docs.oracle.com/en/java/javase/21/) — référence complète du JDK.
- [OpenJDK](https://openjdk.org/) — implémentation open-source de référence.
- [Baeldung](https://www.baeldung.com/) — tutoriels et explications de qualité sur l'écosystème Java/Spring.
- [Maven — Guide de démarrage](https://maven.apache.org/guides/getting-started/) et [Gradle — Documentation](https://docs.gradle.org/current/userguide/userguide.html).
- [roadmap.sh — Java](https://roadmap.sh/java) — vue d'ensemble du parcours d'apprentissage.
