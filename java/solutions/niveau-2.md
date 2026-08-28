# Solutions — Niveau 2

## 2.1 — Interface et polymorphisme

```java
public interface Shape {
    double area();
}

public class Circle implements Shape {
    private final double radius;
    public Circle(double radius) { this.radius = radius; }
    @Override
    public double area() { return Math.PI * radius * radius; }
}

public class Rectangle implements Shape {
    private final double width, height;
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    @Override
    public double area() { return width * height; }
}

public static void printAreas(List<Shape> shapes) {
    for (Shape shape : shapes) {
        System.out.println(shape.area()); // appelle la bonne implémentation selon le type réel, sans if/instanceof
    }
}
```

## 2.2 — Streams

```java
List<String> names = List.of("Alice", "Bob", "Charlotte", "Max");

List<String> upper = names.stream().map(String::toUpperCase).toList();

List<String> longNames = names.stream().filter(n -> n.length() > 4).toList();

int totalChars = names.stream().mapToInt(String::length).sum();
```

## 2.3 — Exception checked

```java
public static int parsePositiveInt(String input) {
    int value = Integer.parseInt(input); // NumberFormatException si invalide (unchecked, pas de throws requis)
    if (value < 0) throw new IllegalArgumentException("doit être positif");
    return value;
}

public class InvalidAgeException extends Exception { // checked : hérite de Exception, pas de RuntimeException
    public InvalidAgeException(String message) {
        super(message);
    }
}

public static void validateAge(int age) throws InvalidAgeException {
    if (age < 0 || age > 150) {
        throw new InvalidAgeException("Âge invalide : " + age);
    }
}

// Appel : le compilateur FORCE ce try/catch (ou une déclaration throws en cascade)
try {
    validateAge(-5);
} catch (InvalidAgeException e) {
    System.out.println(e.getMessage());
}
```

## 2.4 — Map et agrégation

```java
List<String> words = List.of("chat", "chien", "chat", "oiseau", "chien", "chat");

Map<String, Integer> counts = new HashMap<>();
for (String word : words) {
    counts.merge(word, 1, Integer::sum); // incrémente si présent, initialise à 1 sinon
}
// équivalent plus verbeux : counts.put(word, counts.getOrDefault(word, 0) + 1);
```

## 2.5 — Generics

```java
public class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

Box<String> stringBox = new Box<>();
stringBox.set("hello");
String s = stringBox.get(); // pas de cast nécessaire, le compilateur connaît le type T=String

Box<Integer> intBox = new Box<>();
intBox.set(42);
// intBox.set("oops"); // erreur de COMPILATION : String n'est pas Integer
```
