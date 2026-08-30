# Niveau 2 — Intermédiaire

## Exercice 2.1 — Interface et polymorphisme

Crée une interface `IShape` avec une méthode `double Area()`. Implémente-la dans deux classes `Circle` (rayon) et `Rectangle` (largeur, hauteur). Écris une méthode `PrintAreas(List<IShape> shapes)` qui affiche l'aire de chaque forme, sans connaître son type concret.

## Exercice 2.2 — LINQ

À partir d'une `List<string>` de noms, produis (avec LINQ, sans boucle `foreach` manuelle) :
1. La liste des noms en majuscules (`Select`).
2. La liste des noms de plus de 4 lettres (`Where`).
3. Le nombre total de caractères cumulés (`Sum`).

## Exercice 2.3 — Exception custom

Écris une méthode `int ParsePositiveInt(string input)` qui lève une `FormatException` si la conversion échoue (via `int.Parse`) ou une `ArgumentException` si la valeur est négative. Écris ensuite une méthode `void ValidateAge(int age)` qui lève une `InvalidAgeException` (exception custom que tu définis, héritant d'`Exception`) si l'âge n'est pas dans `[0, 150]`.

## Exercice 2.4 — Dictionary et agrégation

À partir d'une `List<string>` de mots, construis un `Dictionary<string, int>` qui compte le nombre d'occurrences de chaque mot (sans LINQ, utilise `TryGetValue` ou l'indexeur avec `GetValueOrDefault`).

## Exercice 2.5 — Generics

Écris une classe générique `Box<T>` avec une méthode `void Set(T value)` et `T Get()`. Instancie-la avec `string` puis avec `int` pour vérifier que le typage est respecté à la compilation.
