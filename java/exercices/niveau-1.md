# Niveau 1 — Bases

## Exercice 1.1 — Hello, typé

Écris une classe `Greeter` avec une méthode `main` qui déclare une variable `String name`, une variable `int age`, et affiche `"Bonjour <name>, tu as <age> ans."` avec `System.out.println`.

## Exercice 1.2 — Boxing piège

Sans exécuter le code, prédis le résultat de ce programme, puis vérifie :

```java
Integer a = 100;
Integer b = 100;
Integer c = 200;
Integer d = 200;

System.out.println(a == b);
System.out.println(c == d);
```

Explique en une phrase pourquoi les deux résultats diffèrent.

## Exercice 1.3 — Tableau et boucle

Déclare un tableau `int[] scores` contenant 5 notes. Calcule et affiche leur moyenne avec une boucle `for` classique (sans Stream).

## Exercice 1.4 — StringBuilder

Écris une méthode `String repeat(String word, int times)` qui répète `word` `times` fois séparés par un espace, en utilisant un `StringBuilder` (pas de concaténation `+` en boucle).

## Exercice 1.5 — Switch expression

Écris une méthode `String dayType(int day)` qui retourne `"Week-end"` pour 6 et 7, `"Semaine"` pour les autres, en utilisant la forme moderne `switch (day) { case ... -> ... }`.
