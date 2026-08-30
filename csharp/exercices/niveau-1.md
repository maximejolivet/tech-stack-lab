# Niveau 1 — Bases

## Exercice 1.1 — Hello, typé

Écris un programme (top-level statements) qui déclare une variable `string name`, une variable `int age`, et affiche `"Bonjour <name>, tu as <age> ans."` avec une interpolation de chaîne (`$"..."`, pas de concaténation `+`).

## Exercice 1.2 — Nullable piège

Sans exécuter le code, prédis le résultat de chaque ligne, puis vérifie :

```csharp
string? nickname = null;

Console.WriteLine(nickname?.Length);
Console.WriteLine(nickname?.Length ?? -1);
Console.WriteLine(nickname?.ToUpper()?.Length ?? 0);
```

Explique en une phrase la différence entre `?.` et `??`.

## Exercice 1.3 — Liste et boucle

Déclare une `List<int> scores` contenant 5 notes. Calcule et affiche leur moyenne avec une boucle `foreach` classique (sans LINQ).

## Exercice 1.4 — Switch expression

Écris une méthode `string DayType(int day)` qui retourne `"Week-end"` pour 6 et 7, `"Semaine"` pour 1 à 5, `"Invalide"` sinon, en utilisant un `switch` **expression** avec des patterns d'intervalle (`>= 1 and <= 5`).

## Exercice 1.5 — Méthode avec paramètre par défaut

Écris une méthode `string Greet(string name, string greeting = "Bonjour")` et appelle-la trois fois : sans deuxième argument, avec un deuxième argument positionnel, puis avec des arguments nommés dans l'ordre inverse (`Greet(greeting: "Hey", name: "Max")`).
