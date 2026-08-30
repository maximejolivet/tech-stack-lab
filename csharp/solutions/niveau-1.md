# Solutions — Niveau 1

## 1.1 — Hello, typé

```csharp
string name = "Max";
int age = 30;
Console.WriteLine($"Bonjour {name}, tu as {age} ans.");
```

## 1.2 — Nullable piège

```csharp
Console.WriteLine(nickname?.Length);                          // (rien, ligne vide) : null-conditional sur null → null
Console.WriteLine(nickname?.Length ?? -1);                      // -1 : null-coalescing fournit la valeur par défaut
Console.WriteLine(nickname?.ToUpper()?.Length ?? 0);              // 0 : chaîne de null-conditional, s'arrête au premier null
```

`?.` (null-conditional) propage `null` immédiatement si l'objet est `null` au lieu de lever une `NullReferenceException` ; `??` (null-coalescing) fournit une valeur de remplacement quand l'expression à sa gauche vaut `null`.

## 1.3 — Liste et boucle

```csharp
List<int> scores = new List<int> { 12, 15, 8, 18, 10 };
int sum = 0;
foreach (int score in scores)
{
    sum += score;
}
double average = (double)sum / scores.Count; // cast obligatoire pour une division flottante
Console.WriteLine($"Moyenne : {average}");
```

## 1.4 — Switch expression

```csharp
string DayType(int day) => day switch
{
    6 or 7 => "Week-end",
    >= 1 and <= 5 => "Semaine",
    _ => "Invalide"
};
```

## 1.5 — Méthode avec paramètre par défaut

```csharp
string Greet(string name, string greeting = "Bonjour") => $"{greeting}, {name} !";

Console.WriteLine(Greet("Max"));                          // "Bonjour, Max !"
Console.WriteLine(Greet("Max", "Salut"));                   // "Salut, Max !"
Console.WriteLine(Greet(greeting: "Hey", name: "Max"));       // "Hey, Max !" — ordre libre grâce aux arguments nommés
```
