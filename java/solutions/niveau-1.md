# Solutions — Niveau 1

## 1.1 — Hello, typé

```java
public class Greeter {
    public static void main(String[] args) {
        String name = "Max";
        int age = 30;
        System.out.println("Bonjour " + name + ", tu as " + age + " ans.");
    }
}
```

## 1.2 — Boxing piège

```java
System.out.println(a == b); // true
System.out.println(c == d); // false
```

La JVM met en cache les objets `Integer` pour les valeurs entre -128 et 127 (`IntegerCache`). `a` et `b` (100) pointent donc vers le même objet caché → `==` compare la même référence → `true`. `c` et `d` (200) sont hors du cache : deux objets distincts sont créés → `==` compare des références différentes → `false`. C'est pour cette raison qu'il faut toujours utiliser `.equals()` pour comparer des valeurs de wrappers.

## 1.3 — Tableau et boucle

```java
int[] scores = {12, 15, 8, 18, 10};
int sum = 0;
for (int i = 0; i < scores.length; i++) {
    sum += scores[i];
}
double average = (double) sum / scores.length; // cast obligatoire pour une division flottante
System.out.println("Moyenne : " + average);
```

## 1.4 — StringBuilder

```java
public static String repeat(String word, int times) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < times; i++) {
        if (i > 0) sb.append(" ");
        sb.append(word);
    }
    return sb.toString();
}
```

## 1.5 — Switch expression

```java
public static String dayType(int day) {
    return switch (day) {
        case 6, 7 -> "Week-end";
        default -> "Semaine";
    };
}
```
