# Solutions — Niveau 3

## 3.1 — Records et pattern matching exhaustif

```csharp
public abstract record PaymentEvent;
public record PaymentSucceeded(string OrderId, double Amount) : PaymentEvent;
public record PaymentFailed(string OrderId, string Reason) : PaymentEvent;

public string Describe(PaymentEvent evt) => evt switch
{
    PaymentSucceeded s => $"Commande {s.OrderId} payée : {s.Amount}€",
    PaymentFailed f => $"Commande {f.OrderId} échouée : {f.Reason}",
    _ => throw new ArgumentException("Type d'événement inconnu")
    // contrairement au sealed interface + switch exhaustif de Java/Kotlin, C# ne peut pas garantir
    // statiquement la couverture de tous les sous-types d'un record abstrait : le "_" reste nécessaire
};
```

## 3.2 — Concurrence avec async/await et Task.WhenAll

```csharp
async Task<int> ProcessTaskAsync(int id)
{
    await Task.Delay(500); // simule un traitement (ex: appel réseau), sans bloquer le thread
    return id * 2;
}

var start = DateTime.Now;

var tasks = Enumerable.Range(0, 10).Select(ProcessTaskAsync).ToList();
var results = await Task.WhenAll(tasks); // attend que TOUTES les tâches se terminent, en parallèle

Console.WriteLine($"Durée totale : {(DateTime.Now - start).TotalMilliseconds}ms");
// Task.WhenAll lance les 10 tâches concurremment sur le thread pool : temps total proche de 500ms
// Un traitement séquentiel (await un par un dans une boucle) aurait pris 10 * 500ms = 5000ms
```

## 3.3 — Générations du garbage collector

```csharp
public class Cache
{
    // Fuite potentielle : cette liste static grandit indéfiniment sur toute la durée de vie
    // de l'application, aucune entrée n'est jamais retirée (pas de TTL, pas de taille max).
    private static readonly List<byte[]> _cache = new();

    public static void Store(byte[] data)
    {
        _cache.Add(data); // jamais de Remove() correspondant → croissance illimitée du heap
    }
}
```

Le GC du CLR classe les objets par génération selon leur durée de vie observée : un objet nouvellement alloué démarre en Gen0, collectée très fréquemment et à faible coût. S'il survit à une collection Gen0 (parce qu'une référence l'atteint encore, comme ici depuis `_cache`), il est promu en Gen1, puis en Gen2 s'il survit encore. Les objets accumulés dans `_cache` ne sont jamais éligibles à la collecte (la référence `static` les atteint toujours) : ils finissent tous promus en Gen2, dont les collections sont bien plus coûteuses (elle scanne une portion beaucoup plus large du heap) — la fuite dégrade donc directement la performance de toute l'application, pas seulement sa consommation mémoire.

## 3.4 — Design orienté interfaces

```csharp
public interface IUserRepository
{
    User? FindById(string id);
}

public class SqlUserRepository : IUserRepository
{
    public User? FindById(string id)
    {
        // requête SQL réelle
        return null;
    }
}

public class InMemoryUserRepository : IUserRepository
{
    private readonly Dictionary<string, User> _users = new();
    public User? FindById(string id) => _users.GetValueOrDefault(id);
}

public class UserService
{
    private readonly IUserRepository _repository; // dépend de l'interface, jamais de SqlUserRepository directement
    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
}
```

Dependency Inversion : `UserService` (module de haut niveau) ne dépend pas de `SqlUserRepository` (détail concret bas niveau) mais de l'abstraction `IUserRepository` — on peut injecter `InMemoryUserRepository` en test sans toucher au code métier, exactement le même principe que l'injection de dépendances par interface pratiquée en Java/Kotlin/Symfony/Laravel, et natif au conteneur DI d'ASP.NET Core.
