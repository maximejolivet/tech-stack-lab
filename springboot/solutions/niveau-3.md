# Spring Boot — Solutions niveau 3

## Exercice 1

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    private final OrderService service;
    public OrderController(OrderService service) { this.service = service; }

    @PostMapping
    public ResponseEntity<Order> create(@Valid @RequestBody CreateOrderRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.create(request));
    }
}

@Service
public class OrderService {
    private final OrderRepository repository;
    public OrderService(OrderRepository repository) { this.repository = repository; }

    @Transactional
    public Order create(CreateOrderRequest request) {
        if (request.items().isEmpty()) {
            throw new InvalidOrderException("Une commande doit contenir au moins un article");
        }
        Order order = new Order();
        // mapping des lignes...
        return repository.save(order);
    }
}
```

**Pourquoi ça compte** : un "fat controller" qui contiendrait directement la règle `if (items.isEmpty())` ne peut pas être testé sans démarrer une requête HTTP complète, ne peut pas être réutilisé (ex. depuis un job batch qui crée des commandes), et mélange la responsabilité HTTP (codes de statut, sérialisation) avec la responsabilité métier (règles de validité d'une commande). Séparer permet de tester `OrderService.create()` unitairement, avec un `OrderRepository` mocké.

## Exercice 2

```java
@GetMapping
public Page<Order> findAll(@RequestParam(defaultValue = "0") int page,
                             @RequestParam(defaultValue = "20") int size) {
    return repository.findAll(PageRequest.of(page, size));
}
```

Sans pagination, `findAll()` sur une table de plusieurs millions de lignes charge l'intégralité des données en mémoire côté application et les sérialise en JSON en une seule réponse — latence énorme, risque d'`OutOfMemoryError`, et bande passante gaspillée côté client qui n'affichera de toute façon qu'une page à la fois. `Pageable` traduit la pagination en `LIMIT`/`OFFSET` SQL, ne charge que les lignes nécessaires.

## Exercice 3

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
            .requestMatchers(HttpMethod.GET, "/api/orders/**").permitAll()
            .anyRequest().authenticated()
        ).httpBasic(Customizer.withDefaults());
        return http.build();
    }
}
```

**Authentification** = vérifier l'identité (qui es-tu ? via `httpBasic`, un token JWT, etc.) — gérée ici par la chaîne de filtres avant que la requête n'atteigne le contrôleur. **Autorisation** = vérifier les droits une fois l'identité connue (as-tu le droit de faire cette action ?) — ici simplifiée à "authentifié ou non", mais pourrait être affinée avec des rôles (`hasRole("ADMIN")`) pour distinguer par exemple qui peut supprimer une commande.

## Exercice 4

**Cause probable** : la liste des commandes charge une relation `@OneToMany` (les lignes d'articles) en lazy loading, et le code accède à cette collection pour chaque commande dans une boucle (ex. dans le mapping vers un DTO) — une requête SQL supplémentaire par commande.

**Correction** :

```java
@Query("SELECT DISTINCT o FROM Order o JOIN FETCH o.items WHERE ...")
List<Order> findAllWithItems();
```

ou avec `@EntityGraph(attributePaths = "items")` sur la méthode du repository.

**Détection plus tôt** : activer `spring.jpa.show-sql=true` (et `spring.jpa.properties.hibernate.format_sql=true`) en dev pour voir le nombre réel de requêtes générées, ou utiliser un outil comme p6spy / le plugin Hibernate Statistics qui compte les requêtes par transaction — à intégrer idéalement dans les tests d'intégration (assertion sur le nombre de requêtes) pour éviter une régression silencieuse en production.
