# Niveau 2 — Intermédiaire

## Exercice 2.1 — Interface et polymorphisme

Crée une interface `Shape` avec une méthode `double area()`. Implémente-la dans deux classes `Circle` (rayon) et `Rectangle` (largeur, hauteur). Écris une méthode `printAreas(List<Shape> shapes)` qui affiche l'aire de chaque forme, sans connaître son type concret.

## Exercice 2.2 — Streams

À partir d'une `List<String>` de noms, produis (avec des Streams, sans boucle `for` manuelle) :
1. La liste des noms en majuscules.
2. La liste des noms de plus de 4 lettres.
3. Le nombre total de caractères cumulés (`reduce` ou `mapToInt().sum()`).

## Exercice 2.3 — Exception checked

Écris une méthode `int parsePositiveInt(String input) throws NumberFormatException` (unchecked) puis une méthode `void validateAge(int age) throws InvalidAgeException` où `InvalidAgeException` est une exception **checked** que tu définis toi-même (hérite de `Exception`). Le code appelant doit obligatoirement gérer cette exception.

## Exercice 2.4 — Map et agrégation

À partir d'une `List<String>` de mots, construis une `Map<String, Integer>` qui compte le nombre d'occurrences de chaque mot (utilise `getOrDefault` ou `Map.merge`).

## Exercice 2.5 — Generics

Écris une classe générique `Box<T>` avec une méthode `void set(T value)` et `T get()`. Instancie-la avec `String` puis avec `Integer` pour vérifier que le typage est respecté à la compilation.
