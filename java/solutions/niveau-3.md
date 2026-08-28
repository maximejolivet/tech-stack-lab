# Solutions — Niveau 3

## 3.1 — Records et sealed classes

```java
public sealed interface PaymentEvent permits PaymentSucceeded, PaymentFailed {}

public record PaymentSucceeded(String orderId, double amount) implements PaymentEvent {}
public record PaymentFailed(String orderId, String reason) implements PaymentEvent {}

public static String describe(PaymentEvent event) {
    return switch (event) {
        case PaymentSucceeded s -> "Commande " + s.orderId() + " payée : " + s.amount() + "€";
        case PaymentFailed f -> "Commande " + f.orderId() + " échouée : " + f.reason();
        // pas de default nécessaire : sealed + toutes les implémentations couvertes
        // = le compilateur refuse de compiler si un cas manque
    };
}
```

## 3.2 — Concurrence avec ExecutorService

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
List<Future<Integer>> futures = new ArrayList<>();

long start = System.currentTimeMillis();

for (int i = 0; i < 10; i++) {
    int taskId = i;
    futures.add(executor.submit(() -> {
        Thread.sleep(500); // simule un traitement (ex: appel réseau)
        return taskId * 2;
    }));
}

for (Future<Integer> future : futures) {
    System.out.println(future.get()); // bloque jusqu'au résultat de CHAQUE tâche
}

executor.shutdown();
System.out.println("Durée totale : " + (System.currentTimeMillis() - start) + "ms");
// Avec 4 threads : ~3 vagues de traitement (10 tâches / 4 threads) → environ 1500ms
// Un traitement séquentiel aurait pris 10 * 500ms = 5000ms
```

## 3.3 — Gestion mémoire et fuite simulée

```java
public class Cache {
    // Fuite potentielle : cette liste static grandit indéfiniment sur toute la durée de vie
    // de l'application, aucune entrée n'est jamais retirée (pas de TTL, pas de taille max).
    private static final List<byte[]> cache = new ArrayList<>();

    public static void store(byte[] data) {
        cache.add(data); // jamais de remove() correspondant → croissance illimitée du heap
    }
}
```

Dans une app long-running (JVM qui tourne des jours/semaines), toute référence conservée indéfiniment dans une structure `static` ou un singleton empêche le garbage collector de libérer la mémoire correspondante — le heap grossit jusqu'à un `OutOfMemoryError`. En PHP classique, ce risque n'existe pas de la même façon car le process (et toute sa mémoire) est détruit à la fin de chaque requête ; il redevient pertinent dès qu'on passe à un runtime PHP persistant (FrankenPHP en mode worker, Swoole) — même classe de bug que Node.js ou Java.

## 3.4 — Design orienté interfaces

```java
public interface UserRepository {
    Optional<User> findById(String id);
}

public class MySQLUserRepository implements UserRepository {
    @Override
    public Optional<User> findById(String id) {
        // requête SQL réelle
        return Optional.empty();
    }
}

public class InMemoryUserRepository implements UserRepository {
    private final Map<String, User> users = new HashMap<>();
    @Override
    public Optional<User> findById(String id) {
        return Optional.ofNullable(users.get(id));
    }
}

public class UserService {
    private final UserRepository repository; // dépend de l'interface, jamais de MySQLUserRepository directement
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Dependency Inversion : `UserService` (module de haut niveau) ne dépend pas de `MySQLUserRepository` (détail concret bas niveau) mais de l'abstraction `UserRepository` — on peut injecter `InMemoryUserRepository` en test sans toucher au code métier, exactement le même principe que l'injection de dépendances par interface pratiquée en Symfony/Laravel.
