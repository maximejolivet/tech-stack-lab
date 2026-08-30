# Solutions — Niveau 2

## 2.1 — Interface et polymorphisme

```csharp
public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    private readonly double _radius;
    public Circle(double radius) => _radius = radius;
    public double Area() => Math.PI * _radius * _radius;
}

public class Rectangle : IShape
{
    private readonly double _width, _height;
    public Rectangle(double width, double height)
    {
        _width = width;
        _height = height;
    }
    public double Area() => _width * _height;
}

public void PrintAreas(List<IShape> shapes)
{
    foreach (var shape in shapes)
    {
        Console.WriteLine(shape.Area()); // appelle la bonne implémentation selon le type réel, sans if/is
    }
}
```

## 2.2 — LINQ

```csharp
var names = new List<string> { "Alice", "Bob", "Charlotte", "Max" };

var upper = names.Select(n => n.ToUpper()).ToList();

var longNames = names.Where(n => n.Length > 4).ToList();

int totalChars = names.Sum(n => n.Length);
```

## 2.3 — Exception custom

```csharp
public int ParsePositiveInt(string input)
{
    int value = int.Parse(input); // lève FormatException si invalide
    if (value < 0) throw new ArgumentException("doit être positif");
    return value;
}

public class InvalidAgeException : Exception
{
    public InvalidAgeException(string message) : base(message) { }
}

public void ValidateAge(int age)
{
    if (age < 0 || age > 150)
    {
        throw new InvalidAgeException($"Âge invalide : {age}");
    }
}

// Appel : contrairement à Java, aucune déclaration "throws" n'est requise, ni côté méthode ni côté appelant
try
{
    ValidateAge(-5);
}
catch (InvalidAgeException e)
{
    Console.WriteLine(e.Message);
}
```

## 2.4 — Dictionary et agrégation

```csharp
var words = new List<string> { "chat", "chien", "chat", "oiseau", "chien", "chat" };

var counts = new Dictionary<string, int>();
foreach (var word in words)
{
    counts[word] = counts.GetValueOrDefault(word, 0) + 1; // incrémente si présent, initialise à 1 sinon
}
```

## 2.5 — Generics

```csharp
public class Box<T>
{
    private T _value = default!;
    public void Set(T value) => _value = value;
    public T Get() => _value;
}

var stringBox = new Box<string>();
stringBox.Set("hello");
string s = stringBox.Get(); // pas de cast nécessaire, le compilateur connaît le type T=string

var intBox = new Box<int>();
intBox.Set(42);
// intBox.Set("oops"); // erreur de COMPILATION : string n'est pas int
```
