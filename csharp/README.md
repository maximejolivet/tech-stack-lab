# C#

## 1. Introduction

C# est un langage compilé, orienté objet, à typage statique fort, qui s'exécute sur le CLR (Common Language Runtime) — l'équivalent .NET de la JVM déjà couverte via [`../java/`](../java/) et [`../kotlin/`](../kotlin/) : compilation en bytecode intermédiaire (IL, Intermediate Language) puis exécution/JIT par le runtime. Ce dossier couvre le **langage C#** et les bases d'**ASP.NET Core** (le framework web officiel de l'écosystème .NET) ensemble, faute de dossier dédié séparé — même logique que [`../flutter/`](../flutter/) (Dart + Flutter) ou [`../swift/`](../swift/) (Swift + SwiftUI).

**À quoi sert-il ?**
- Applications backend d'entreprise, historiquement sur Windows/IIS, aujourd'hui multiplateforme (Linux/macOS/Docker) depuis .NET Core (2016) — écosystème très présent dans les grandes entreprises et le secteur bancaire/assurance.
- APIs web et microservices via ASP.NET Core, avec des performances parmi les meilleures des frameworks web mainstream (benchmarks TechEmpower).
- Développement de jeux vidéo via Unity, qui utilise C# comme langage de scripting officiel.

**Où se situe-t-il dans une architecture web ?**
Côté serveur, dans les mêmes conditions qu'une application Java ou Kotlin : process .NET long-running (Kestrel, le serveur web intégré à ASP.NET Core), pas de modèle "process tué à chaque requête" comme PHP-FPM. Kestrel peut être exposé directement ou derrière un reverse proxy (Nginx, IIS en mode reverse proxy).

**Avantages**
- Typage statique fort avec une inférence de type très complète (`var`), erreurs de type détectées à la compilation — même philosophie que Java/Kotlin.
- Null safety intégrée au système de types depuis C# 8 (*nullable reference types*), comparable directement à ce que Kotlin ([`../kotlin/`](../kotlin/)) et Swift ([`../swift/`](../swift/)) proposent déjà dans ce repo.
- Performance native forte sur ASP.NET Core (compilation AOT possible depuis .NET 7+), et une seule stack officielle cohérente (langage, runtime, framework web, ORM) maintenue par Microsoft plutôt qu'un patchwork de projets communautaires indépendants.

**Limites**
- Historiquement perçu (à tort aujourd'hui) comme un écosystème "Windows only" — l'héritage culturel freine encore son adoption dans certains milieux full-stack web/PHP/JS.
- Verbosité proche de Java sur du code non idiomatique (bien que records, pattern matching et top-level statements la réduisent nettement depuis C# 9-10).
- Cycle compile → package → déployer plus lourd qu'un langage interprété comme PHP ou Python, même limite structurelle que Java/Kotlin.

## 2. Prérequis

- POO déjà pratiquée en PHP, Java ou Kotlin ([`../php/`](../php/), [`../java/`](../java/), [`../kotlin/`](../kotlin/)) — classes, interfaces, héritage sont directement transférables.
- .NET SDK installé en local (`dotnet --version`) — .NET 8 (LTS) recommandé.
- Un IDE avec support C# (Visual Studio sur Windows, JetBrains Rider multiplateforme, ou VS Code + extension C# Dev Kit) plutôt qu'un simple éditeur de texte.

## 3. Rappel des bases 🟢

### 01 - Le CLR et la compilation

**Explication** — Le code `.cs` est compilé par le compilateur Roslyn en **IL** (Intermediate Language, l'équivalent .NET du bytecode Java), exécuté par le CLR qui le compile à son tour en code machine natif via son propre JIT — même modèle en deux étapes que la JVM, avec un runtime différent.

```bash
dotnet build          # compile le projet en assemblies IL (.dll)
dotnet run              # compile si nécessaire, puis exécute
```

**Cas d'usage** : comme pour Java/Kotlin, cette étape de compilation intermédiaire permet à un même exécutable managé de cibler plusieurs OS (Windows/Linux/macOS) tant que le runtime .NET correspondant est installé.

### 02 - Structure minimale et top-level statements

**Explication** — Depuis C# 9, un point d'entrée n'a plus besoin d'être enveloppé dans une classe avec une méthode `Main` explicite : des instructions au niveau fichier suffisent (le compilateur génère la classe et la méthode `Main` en interne).

```csharp
// Program.cs
Console.WriteLine("Hello, world!");
```

**Erreur fréquente** : chercher `public static void Main` par réflexe Java — cette syntaxe reste valide et nécessaire dès qu'on a besoin d'arguments (`string[] args`) ou de plusieurs fichiers avec du code de haut niveau, mais n'est plus obligatoire pour un point d'entrée simple.

### 03 - var et inférence de type

**Explication** — `var` infère le type à la compilation à partir de l'expression assignée ; le type reste **statique** une fois déterminé — ce n'est pas un `var` JS dynamique, exactement la même sémantique que le `var` Java 10+.

```csharp
var name = "Max";      // inféré string, reste string pour toujours
var age = 30;            // inféré int
string city = "Paris";     // typage explicite, équivalent

// age = "trente";        // erreur de COMPILATION : impossible d'assigner un autre type
```

**Bonne pratique** : typer explicitement les signatures publiques (paramètres, retours) même avec `var` disponible pour les variables locales — la clarté de l'API prime, même recommandation qu'en Java.

### 04 - Nullable reference types (null safety)

**Explication** — Depuis C# 8, avec `#nullable enable` (activé par défaut dans les nouveaux projets), `string` ne peut plus contenir `null` sans l'annotation `?` — le compilateur émet un avertissement si une valeur potentiellement nulle est utilisée sans vérification. Comparaison directe avec le `String?` de Kotlin ([`../kotlin/`](../kotlin/)) et l'`Optional` de Swift ([`../swift/`](../swift/)).

```csharp
string name = "Max";        // ne devrait jamais être null
string? nickname = null;      // explicitement nullable

Console.WriteLine(nickname?.Length);        // null-conditional : null si nickname est null
Console.WriteLine(nickname?.Length ?? 0);    // null-coalescing : valeur par défaut si null
Console.WriteLine(nickname!.Length);          // "null-forgiving" : force le compilateur à faire confiance, crash si null
```

**Erreur fréquente** : abuser de l'opérateur `!` (null-forgiving) pour faire taire un avertissement du compilateur sans traiter le cas `null` — contrairement à Kotlin où `!!` lève systématiquement une exception au runtime, `!` en C# ne fait que supprimer l'avertissement de compilation ; le risque de `NullReferenceException` reste entier si la valeur est réellement `null`.

### 05 - Value types vs reference types (struct vs class)

**Explication** — C# distingue explicitement `struct` (type valeur, copié à l'affectation) et `class` (type référence, partagé par pointeur) — exactement la même distinction que les `struct`/`class` de Swift ([`../swift/`](../swift/)).

```csharp
struct Point
{
    public int X, Y;
}

Point p1 = new Point { X = 1, Y = 2 };
Point p2 = p1;       // copie complète, indépendante
p2.X = 99;
Console.WriteLine(p1.X); // 1 — p1 n'est pas affecté, contrairement à une class
```

**Erreur fréquente** : s'attendre à un comportement partagé (référence) sur une `struct` par réflexe Java/Kotlin (où tout objet est une référence) — une `struct` C# copiée n'affecte jamais l'original.

### 06 - Structures de contrôle et pattern matching

**Explication** — `if/else`, `for`, `while` proches de Java. Le `switch` moderne (C# 8+) est une **expression** qui retourne une valeur et supporte le pattern matching, comparable au `when` de Kotlin ou au `switch` expression de Java 14+.

```csharp
int age = 20;
string status = age >= 18 ? "majeur" : "mineur";

string label = dayOfWeek switch
{
    6 or 7 => "Week-end",
    >= 1 and <= 5 => "Semaine",
    _ => "Invalide"
};
```

**Cas d'usage** : le pattern matching sur des intervalles (`>= 1 and <= 5`) et des types (`is Circle c`) rend le `switch` expression bien plus expressif qu'un `switch` classique C-style à base de `case: break;`.

### 07 - Collections de base

**Explication** — `List<T>` (liste dynamique typée, équivalent d'`ArrayList<T>` en Java ou `MutableList<T>` en Kotlin), `Dictionary<TKey, TValue>` (clé-valeur, équivalent de `HashMap`/`Map`).

```csharp
var scores = new List<int> { 12, 15, 8, 18, 10 };
scores.Add(20);

var ages = new Dictionary<string, int> { ["Alice"] = 30, ["Bob"] = 25 };
Console.WriteLine(ages.GetValueOrDefault("Charlie", 0)); // valeur par défaut si clé absente
```

**Bonne pratique** : préférer `IReadOnlyList<T>`/`IReadOnlyDictionary<K,V>` dans une signature publique quand la collection ne doit pas être modifiée par l'appelant — communique l'intention au même titre que `List`/`MutableList` en Kotlin.

### 08 - Interpolation de chaînes

**Explication** — La syntaxe `$"..."` insère directement des expressions dans une chaîne, comme les string templates Kotlin (`$variable`) ou Swift (`\(expression)`).

```csharp
string name = "Max";
int age = 30;
Console.WriteLine($"Bonjour {name}, tu as {age + 1} ans l'année prochaine.");

string html = $"""
    <html>
        <body>Hello</body>
    </html>
    """; // raw string literal (C# 11+), indentation gérée automatiquement
```

**Erreur fréquente** : utiliser `string.Format()` ou la concaténation `+` par habitude d'un ancien style — l'interpolation `$"..."` est plus lisible et le standard moderne recommandé.

### 09 - Records (types immuables)

**Explication** — Un `record` (C# 9+) génère automatiquement constructeur, propriétés, `Equals()`/`GetHashCode()`/`ToString()`, et une méthode `with` pour créer une copie modifiée — équivalent quasi direct des `record` Java et des `data class` Kotlin déjà vus dans ce repo.

```csharp
public record Point(int X, int Y);

var p1 = new Point(3, 4);
var p2 = p1 with { Y = 10 }; // copie avec Y modifié, p1 reste inchangé
Console.WriteLine(p1); // Point { X = 3, Y = 4 } — ToString() généré automatiquement
```

## 4. Concepts intermédiaires 🟡

- **Propriétés (get/set)** : une propriété expose un champ avec une syntaxe d'accès directe tout en gardant la possibilité d'exécuter du code à la lecture/écriture — pas d'équivalent syntaxique aussi direct en Java (qui utilise des getters/setters explicites).

```csharp
public class User
{
    public string Name { get; set; } = "";
    public int Age { get; private set; } // lecture publique, écriture seulement depuis la classe

    public User(int age) => Age = age;
}
```

- **Interfaces et polymorphisme** : une `interface` définit un contrat, une classe peut en implémenter plusieurs — même principe qu'en Java/Kotlin, avec le support des méthodes d'interface par défaut (*default interface methods*) depuis C# 8.

```csharp
public interface IPaymentMethod
{
    bool Pay(double amount);
}

public class CreditCard : IPaymentMethod
{
    public bool Pay(double amount)
    {
        // logique de paiement
        return true;
    }
}
```

- **Génériques** : `List<T>` plutôt qu'une collection non typée — le compilateur garantit le type contenu, éliminant une classe d'`InvalidCastException` runtime, exactement le même bénéfice qu'en Java/Kotlin.
- **Async/await (Task-based)** : modèle de concurrence asynchrone basé sur `Task`/`Task<T>`, syntaxiquement très proche des coroutines Kotlin ([`../kotlin/`](../kotlin/)) ou des promesses JS — `await` suspend sans bloquer le thread appelant.

```csharp
public async Task<string> FetchUserNameAsync()
{
    await Task.Delay(500); // suspend sans bloquer le thread, contrairement à Thread.Sleep
    return "Max";
}
```

- **Gestion des exceptions** : `try/catch/finally`, hiérarchie d'exceptions natives (`ArgumentException`, `InvalidOperationException`...), exceptions custom héritant d'`Exception`. Contrairement à Java, il n'existe **aucune distinction checked/unchecked** en C# — toute exception se propage librement, sans déclaration obligatoire (comportement identique à Python/Kotlin sur ce point précis).

```csharp
public class InsufficientFundsException : Exception
{
    public InsufficientFundsException(string message) : base(message) { }
}

try
{
    throw new InsufficientFundsException("Solde insuffisant");
}
catch (InsufficientFundsException e)
{
    Console.WriteLine(e.Message);
}
```

- **LINQ (Language Integrated Query)** : programmation fonctionnelle déclarative sur des collections, la fonctionnalité la plus signature de C# — comparable aux Streams Java ou aux fonctions de collection Kotlin (`map`/`filter`), avec en plus une syntaxe façon requête SQL en option (*query syntax*).

```csharp
var prices = new List<int> { 10, 25, 8, 42 };

// Method syntax (la plus courante)
var withTax = prices.Where(p => p > 20).Select(p => (int)(p * 1.2)).ToList();

// Query syntax (équivalente, plus rare en pratique)
var withTax2 = (from p in prices where p > 20 select (int)(p * 1.2)).ToList();
```

**Erreur fréquente** : oublier que la plupart des opérateurs LINQ (`Where`, `Select`) sont **paresseux** (deferred execution) — la requête n'est réellement exécutée qu'au moment de son énumération (`ToList()`, `foreach`), pas à sa déclaration ; itérer plusieurs fois une même requête LINQ non matérialisée la ré-exécute à chaque fois.

- **ASP.NET Core — Minimal API** : depuis .NET 6, une API HTTP minimale peut être déclarée sans contrôleur, avec l'injection de dépendances intégrée au conteneur natif du framework.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IUserRepository, InMemoryUserRepository>(); // DI native

var app = builder.Build();

app.MapGet("/users/{id}", (int id, IUserRepository repo) => repo.FindById(id));

app.Run();
```

## 5. Concepts avancés 🟠🔴

- **Garbage collector et générations** : le CLR utilise un GC générationnel (Gen0/Gen1/Gen2) conceptuellement proche du GC de la JVM déjà couvert dans [`../java/`](../java/) — les objets récents (Gen0) sont collectés fréquemment et rapidement, les objets qui survivent plusieurs cycles migrent vers Gen1 puis Gen2, collectées plus rarement. Une app .NET long-running accumule un état mémoire sur toute sa durée de vie, exactement le même risque de fuite que pour Java/Node.js.
- **Pattern matching avancé (recursive patterns)** : le `switch` expression peut déstructurer des objets et des `record` en profondeur, combiné à des gardes (`when`), pour exprimer des conditions complexes de façon déclarative plutôt qu'en cascade d'`if`.

```csharp
public record Order(string Status, double Amount);

string Describe(Order order) => order switch
{
    { Status: "Paid", Amount: > 1000 } => "Grosse commande payée",
    { Status: "Paid" } => "Commande payée",
    { Status: "Cancelled" } => "Commande annulée",
    _ => "Statut inconnu"
};
```

- **Span\<T\> et Memory\<T\>** : types permettant de manipuler des segments contigus de mémoire (tableaux, chaînes) **sans allocation supplémentaire** ni copie — utilisés dans les chemins de code critiques en performance (parsing, sérialisation) où Java/Kotlin n'ont pas d'équivalent direct aussi intégré au langage.
- **Middleware et conteneur DI d'ASP.NET Core en profondeur** : chaque requête traverse un pipeline de middlewares configurables (authentification, logging, gestion d'erreurs) ; le conteneur d'injection de dépendances natif gère trois durées de vie (`Singleton`, `Scoped` — une instance par requête HTTP, `Transient` — une nouvelle instance à chaque résolution), un concept comparable aux scopes de services Symfony mais intégré au cœur du framework plutôt qu'en configuration YAML.
- **Entity Framework Core** : ORM officiel de l'écosystème .NET, conceptuellement comparable à Doctrine ([`../symfony/`](../symfony/)) ou Eloquent ([`../laravel/`](../laravel/)) — migrations basées sur le code (`dotnet ef migrations add`), requêtes exprimées directement en LINQ contre les `DbSet<T>` plutôt qu'un langage de requête séparé.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
}

var adults = context.Users.Where(u => u.Age >= 18).ToList(); // LINQ traduit en SQL par EF Core
```

## 6. Commandes / syntaxe à connaître

```bash
dotnet --version                    # version du SDK installé
dotnet new console -o MonApp          # créer un nouveau projet console
dotnet new webapi -o MonApi             # créer un nouveau projet ASP.NET Core Web API
dotnet restore                            # restaurer les dépendances NuGet
dotnet build                                # compiler le projet
dotnet run                                    # compiler (si besoin) et exécuter
dotnet test                                     # lancer les tests
dotnet publish -c Release                         # build de production
```

```csharp
// Syntaxe essentielle à avoir sous les doigts
var list = new List<int> { 1, 2, 3 }.Where(x => x > 1).ToList();
public record Point(int X, int Y);
var result = nickname?.ToUpper() ?? "ANONYME";
var label = value switch { int n => "nombre", string s => "texte", _ => "autre" };
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Gestionnaire de bibliothèque en CLI, orienté objet**

Reprendre le mini-projet Java ([`../java/README.md`](../java/README.md)) en C# idiomatique, pour comparer directement les deux langages sur le même exercice :
- Modéliser un livre avec un `record Book(string Isbn, string Title, bool Available)`.
- Stocker les livres dans une `List<Book>` en mémoire, avec des opérations via LINQ (recherche par titre, filtrage des disponibles).
- Utiliser une `interface ILibraryAction` implémentée par plusieurs classes (`BorrowAction`, `ReturnAction`) pour illustrer le polymorphisme.
- Gérer une exception métier custom `BookNotFoundException`, levée et catchée proprement (pas de distinction checked/unchecked, contrairement à Java).
- Lire les commandes depuis `Console.ReadLine()` (boucle interactive simple).

Objectif : mobiliser records, interfaces/polymorphisme, LINQ, exceptions et nullable reference types dans le même exercice concret que la version Java, pour comparer directement la verbosité et les garanties du compilateur entre les deux écosystèmes JVM/.NET.

## Checklist

- [ ] Comprendre les fondamentaux (CLR, nullable reference types, value vs reference types)
- [ ] Savoir créer un projet avec le CLI `dotnet`
- [ ] Maîtriser la syntaxe principale (records, pattern matching, LINQ)
- [ ] Comprendre les concepts importants (propriétés, interfaces, async/await, exceptions)
- [ ] Savoir debugger (breakpoints IDE, stack traces)
- [ ] Connaître les bonnes pratiques (éviter `!` sans réflexion, LINQ deferred execution, DI par interface)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (GC générationnel, pattern matching récursif, ASP.NET Core middleware/DI, Entity Framework Core)

## 10. Ressources

- [Documentation officielle C# (Microsoft Learn)](https://learn.microsoft.com/en-us/dotnet/csharp/) — référence complète du langage.
- [Documentation officielle ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/) — référence du framework web.
- [Entity Framework Core — documentation](https://learn.microsoft.com/en-us/ef/core/) pour l'ORM.
- [.NET — Tutoriels interactifs](https://dotnet.microsoft.com/en-us/learn) — parcours guidés officiels.
- [roadmap.sh — ASP.NET Core](https://roadmap.sh/aspnet-core) — vue d'ensemble du parcours d'apprentissage.
