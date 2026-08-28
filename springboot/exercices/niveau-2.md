# Spring Boot — Exercices niveau 2 (Intermédiaire)

## Exercice 1 — Entité et repository

Crée une entité `Product` (`id`, `name`, `price`) avec `@Entity`, et un `ProductRepository extends JpaRepository<Product, Long>`. Ajoute une méthode dérivée `findByNameContainingIgnoreCase(String keyword)`.

## Exercice 2 — Service et injection de dépendances

Crée un `ProductService` qui utilise `ProductRepository` par **injection par constructeur**. Le service expose `findAll()`, `findById(Long id)` (lève une exception custom `ProductNotFoundException` si absent), et `create(Product product)`.

## Exercice 3 — Validation

Crée un DTO `record CreateProductRequest(@NotBlank String name, @Positive double price)`. Le contrôleur `POST /api/products` doit valider ce DTO avec `@Valid` et refuser une requête invalide.

## Exercice 4 — Gestion centralisée des exceptions

Crée un `@RestControllerAdvice` qui intercepte `ProductNotFoundException` (retourne 404) et `MethodArgumentNotValidException` (retourne 400 avec le détail des champs invalides).

## Exercice 5 — Relation

Ajoute une entité `Category` en relation `@ManyToOne` avec `Product`. Écris une requête JPQL avec `JOIN FETCH` pour récupérer tous les produits avec leur catégorie en une seule requête SQL, et explique pourquoi c'est nécessaire.
