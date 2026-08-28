# Spring Boot — Solutions niveau 2

## Exercice 1

```java
@Entity
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;
    // getters/setters
}

public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByNameContainingIgnoreCase(String keyword);
}
```

## Exercice 2

```java
public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(Long id) {
        super("Product not found: " + id);
    }
}

@Service
public class ProductService {
    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    public List<Product> findAll() {
        return repository.findAll();
    }

    public Product findById(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }

    public Product create(Product product) {
        return repository.save(product);
    }
}
```

## Exercice 3

```java
public record CreateProductRequest(
    @NotBlank String name,
    @Positive double price
) {}

@PostMapping
public ResponseEntity<Product> create(@Valid @RequestBody CreateProductRequest request) {
    Product product = new Product();
    product.setName(request.name());
    product.setPrice(request.price());
    return ResponseEntity.status(HttpStatus.CREATED).body(service.create(product));
}
```

Une requête avec `name` vide ou `price` négatif/nul déclenche automatiquement une `MethodArgumentNotValidException` avant d'entrer dans la méthode.

## Exercice 4

```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<Map<String, String>> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(Map.of("error", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(err -> errors.put(err.getField(), err.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

## Exercice 5

```java
@Entity
public class Category {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

```java
@Entity
public class Product {
    // ...
    @ManyToOne
    private Category category;
}
```

```java
@Query("SELECT p FROM Product p JOIN FETCH p.category")
List<Product> findAllWithCategory();
```

Sans `JOIN FETCH`, la relation `@ManyToOne` étant chargée en lazy (ou même en eager par défaut techniquement, mais souvent reconfigurée en lazy), accéder à `product.getCategory().getName()` pour chaque produit dans une boucle déclenche une requête SQL supplémentaire par produit — le classique problème N+1. `JOIN FETCH` charge produits et catégories en une seule requête SQL avec un `JOIN`.
