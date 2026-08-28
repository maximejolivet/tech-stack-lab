# Spring Boot

## 1. Introduction

Spring Boot est le framework Java de référence pour construire des applications backend (API REST, apps web, microservices) en entreprise. Il s'appuie sur **Spring Framework** (Inversion of Control, Dependency Injection) et ajoute une couche de configuration automatique (auto-configuration) qui élimine l'essentiel du XML/boilerplate historique de Spring.

Ce dossier suppose le langage Java déjà acquis ([`../java/`](../java/)) et se concentre uniquement sur ce que le framework ajoute. C'est l'équivalent, côté JVM, de ce que sont [`../symfony/`](../symfony/) et [`../laravel/`](../laravel/) côté PHP : un framework complet (routing, ORM, sécurité, validation) plutôt qu'une simple bibliothèque.

**À quoi sert-il ?**
- Exposer des API REST/GraphQL pour des applications à fort trafic ou fortes contraintes (banque, assurance, grands comptes).
- Construire des microservices (souvent avec Spring Cloud en complément).
- Accéder à des bases de données via un ORM (Spring Data JPA) avec un typage fort de bout en bout.

**Où se situe-t-il dans une architecture web ?**
Côté backend, comme Symfony/Laravel : reçoit une requête HTTP, la route vers un contrôleur, exécute de la logique métier via des services, persiste des données via des repositories, retourne une réponse (JSON en général pour une API).

**Avantages**
- Typage statique fort de bout en bout (contrairement à PHP/JS) → erreurs détectées à la compilation plutôt qu'en production.
- Écosystème Spring très mature (sécurité, batch, cloud, messaging) et standard de facto en entreprise Java.
- Auto-configuration : un projet fonctionnel en quelques minutes, sans XML.

**Limites**
- Verbosité relative comparé à PHP/JS (même si les annotations et Java moderne (records) réduisent l'écart).
- Temps de démarrage JVM plus long que PHP-FPM ou Node (impact sur des architectures serverless "cold start").
- Courbe d'apprentissage de l'IoC container et de "la magie" des annotations, déroutante venant de PHP/JS.

## 2. Prérequis

- Java maîtrisé : POO (classes, interfaces, héritage), collections, exceptions, generics — voir [`../java/`](../java/).
- Maven ou Gradle pour la gestion de dépendances et de build (couvert brièvement dans `../java/`, approfondi ici dans le contexte Spring).
- Notions de base HTTP/REST (méthodes, codes de statut) et SQL utiles pour la partie Spring Data JPA.
- Une IDE avec support Spring est fortement recommandée (IntelliJ IDEA, VS Code + extensions Java).

## 3. Rappel des bases 🟢

### 01 - Spring Initializr et structure d'un projet

**Explication** — [start.spring.io](https://start.spring.io) génère un squelette de projet avec les dépendances choisies (Web, JPA, Security...). Équivalent de `symfony new` / `laravel new`.

```text
src/main/java/com/example/demo/
├── DemoApplication.java       # point d'entrée
├── controller/
├── service/
├── repository/
└── model/
src/main/resources/
├── application.properties     # ou application.yml
└── static/, templates/
src/test/java/...
pom.xml                        # (Maven) ou build.gradle (Gradle)
```

**Bonne pratique** : organiser par couche technique (controller/service/repository) pour un petit projet, par domaine métier (feature packages) pour un gros projet — comme le débat "par type" vs "par feature" en Symfony.

### 02 - Point d'entrée et auto-configuration

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

**Explication** — `@SpringBootApplication` combine trois annotations : `@Configuration` (classe de config), `@EnableAutoConfiguration` (Spring devine la config à partir des dépendances présentes dans le classpath), `@ComponentScan` (scanne le package courant et ses sous-packages pour trouver les composants Spring).

**Erreur fréquente** : déplacer une classe dans un package en dehors de celui de `DemoApplication` → elle n'est plus scannée, donc pas injectée (`NoSuchBeanDefinitionException` au démarrage).

### 03 - Contrôleurs REST

```java
@RestController
@RequestMapping("/api/books")
public class BookController {

    @GetMapping
    public List<Book> findAll() {
        return List.of(new Book(1L, "Clean Code"));
    }

    @GetMapping("/{id}")
    public Book findOne(@PathVariable Long id) {
        return new Book(id, "Clean Code");
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Book create(@RequestBody Book book) {
        return book;
    }
}
```

**Explication** — `@RestController` = `@Controller` + `@ResponseBody` (chaque méthode sérialise directement son retour en JSON, pas de vue à résoudre). `@GetMapping`/`@PostMapping`/etc. sont des raccourcis de `@RequestMapping(method = ...)`.

**Cas d'usage** : équivalent direct d'un contrôleur Symfony avec `#[Route]` ou d'un contrôleur Laravel avec routes définies dans `routes/api.php`.

**Erreur fréquente** : oublier `@RequestBody` sur le paramètre d'un POST → Spring essaie de le lier depuis les query params et reçoit `null`.

### 04 - Configuration (`application.properties` / `.yml`)

```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/demo
    username: demo
    password: ${DB_PASSWORD}
```

**Bonne pratique** : ne jamais committer de secrets en clair — utiliser des variables d'environnement (`${VAR}`) comme dans `.env` chez Symfony/Laravel. Utiliser `application-dev.yml` / `application-prod.yml` avec des profils (voir section intermédiaire) plutôt qu'un seul fichier avec des `if`.

### 05 - Inversion of Control et Application Context

**Explication** — Au démarrage, Spring construit un **Application Context** : un conteneur qui instancie et connecte automatiquement les objets (**beans**) de l'application, à la place du code qui ferait `new` manuellement. C'est le même principe que le Service Container de Symfony ou le Service Container de Laravel — Spring est l'implémentation historique de ce pattern en Java.

**Pourquoi ça existe** : découpler la création des objets de leur utilisation → testabilité (on peut injecter un mock), remplacement d'implémentation sans toucher le code appelant.

## 4. Concepts intermédiaires 🟡

### Dependency Injection

```java
@Service
public class BookService {
    private final BookRepository repository;

    // constructor injection — recommandé
    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public List<Book> findAll() {
        return repository.findAll();
    }
}
```

**Explication** — `@Service`, `@Component`, `@Repository`, `@Controller` marquent une classe comme bean géré par Spring. `@Autowired` peut injecter via un champ ou un setter, mais la **constructor injection** (comme ci-dessus, sans même `@Autowired` si un seul constructeur) est la bonne pratique moderne : elle rend les dépendances obligatoires visibles et immuables (`final`), et facilite les tests unitaires (pas besoin de Spring pour instancier la classe dans un test).

**Erreur fréquente** : injection par champ (`@Autowired private BookRepository repository;`) — fonctionne mais rend la classe difficile à tester sans démarrer tout le contexte Spring, et masque une dépendance circulaire potentielle jusqu'au runtime.

**Comparaison PHP** : équivalent de l'autowiring de Symfony (constructeur + type-hint suffit) ou du binding automatique du Service Container Laravel.

### Spring Data JPA

```java
@Entity
public class Book {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @ManyToOne
    private Author author;
    // getters/setters ou record pour un DTO
}

public interface BookRepository extends JpaRepository<Book, Long> {
    List<Book> findByTitleContaining(String keyword);
}
```

**Explication** — `@Entity` mappe une classe à une table (comme une entité Doctrine ou un model Eloquent). `JpaRepository<Book, Long>` fournit `findAll()`, `save()`, `deleteById()` etc. sans implémentation à écrire ; Spring Data génère l'implémentation à partir du **nom de la méthode** (`findByTitleContaining` → `WHERE title LIKE %?%`).

**Erreur fréquente (le classique N+1)** : itérer sur une liste de `Book` et accéder à `book.getAuthor().getName()` déclenche une requête SQL par livre si la relation est en lazy loading (par défaut pour `@ManyToOne` en théorie eager, mais souvent lazy en pratique pour `@OneToMany`) → utiliser `JOIN FETCH` dans une requête JPQL ou `@EntityGraph` pour charger en une seule requête. Exactement le même piège que le N+1 avec Eloquent ou Doctrine.

**Bonne pratique** : ne jamais exposer directement une entité JPA dans une réponse API (risque de sérialisation infinie sur les relations bidirectionnelles, et couplage API/BDD) — passer par un DTO (souvent un `record` Java).

### Validation

```java
public record CreateBookRequest(
    @NotBlank String title,
    @Min(1) int pages
) {}

@PostMapping
public Book create(@Valid @RequestBody CreateBookRequest request) {
    // si invalide, Spring lève MethodArgumentNotValidException avant d'entrer ici
}
```

**Bonne pratique** : valider en entrée de contrôleur avec Bean Validation (`@NotBlank`, `@Min`, `@Email`...) plutôt qu'à la main dans le service — équivalent des Form Requests Laravel ou des contraintes de validation Symfony.

### Gestion des exceptions

```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(BookNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }
}
```

**Explication** — `@RestControllerAdvice` centralise la gestion des exceptions pour tous les contrôleurs, évitant les `try/catch` répétés — comparable à un `ExceptionListener` Symfony ou un Handler d'exceptions Laravel.

### Profils Spring

```yaml
# application-prod.yml
spring:
  jpa:
    show-sql: false
```

Activation : `--spring.profiles.active=prod`. Équivalent des environnements `APP_ENV=prod` de Symfony/Laravel.

### Tests

```java
@SpringBootTest
@AutoConfigureMockMvc
class BookControllerTest {
    @Autowired MockMvc mockMvc;

    @Test
    void shouldReturnBooks() throws Exception {
        mockMvc.perform(get("/api/books"))
            .andExpect(status().isOk());
    }
}
```

**Bonne pratique** : `@SpringBootTest` démarre tout le contexte (lent, pour tests d'intégration) ; préférer `@WebMvcTest` (contrôleur seul, dépendances mockées) ou des tests unitaires purs (sans Spring du tout) pour la majorité des cas — comme on privilégierait des tests unitaires PHPUnit purs plutôt que des tests fonctionnels Symfony pour tout.

## 5. Concepts avancés 🟠🔴

### Spring Security (bases)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
        return http.build();
    }
}
```

Une chaîne de filtres traite chaque requête (authentification, autorisation) avant d'atteindre le contrôleur — même principe que les firewalls/voters Symfony ou les guards/middlewares Laravel, avec une API plus bas niveau et plus verbeuse.

### Architecture en couches

Controller (HTTP) → Service (logique métier, transactions `@Transactional`) → Repository (accès données). **Règle** : la logique métier ne doit jamais vivre dans le contrôleur ("fat controller" — même anti-pattern qu'en Symfony/Laravel), et un repository ne doit contenir que de l'accès aux données, pas de règles métier.

### Spring WebFlux (programmation réactive) — aperçu

Alternative non-bloquante à Spring MVC (`Mono<T>`/`Flux<T>` au lieu d'objets synchrones), basée sur Reactor. Utile pour un fort volume de requêtes I/O-bound avec peu de threads — conceptuellement proche de l'event loop Node.js, mais avec un modèle de programmation (streams réactifs) nettement plus complexe à maîtriser. À réserver aux cas où le MVC classique (un thread par requête) devient un goulot d'étranglement mesuré, pas par défaut.

### Performance JPA

- **Lazy vs eager loading** : par défaut, préférer le lazy loading et charger explicitement ce qui est nécessaire (`JOIN FETCH`, `@EntityGraph`) plutôt que de tout charger en eager "au cas où".
- **Cache** : cache de premier niveau (session Hibernate, automatique), cache de second niveau (à activer explicitement, ex. avec Redis) pour des données peu volatiles.
- **Pagination** : toujours paginer les listes (`Pageable`) plutôt que `findAll()` sur une table qui grossit.

### Actuator (observabilité)

La dépendance `spring-boot-starter-actuator` expose des endpoints de santé/métriques (`/actuator/health`, `/actuator/metrics`) prêts à l'emploi pour le monitoring en production — sans équivalent standardisé aussi direct côté Symfony/Laravel (où c'est généralement composé à la main ou via des bundles tiers).

### Déploiement

Spring Boot package l'application en **jar exécutable** (serveur Tomcat embarqué inclus) — `java -jar app.jar` suffit à démarrer, pas de configuration serveur externe requise (contrairement à PHP-FPM+Nginx classique, plus proche en philosophie de FrankenPHP). En production : conteneurisation (Docker), gestion du démarrage JVM (taille de heap `-Xmx`), et attention au temps de démarrage pour les déploiements fréquents ou l'autoscaling.

## 6. Commandes / syntaxe à connaître

```bash
# Maven
mvn spring-boot:run          # démarrer en dev
mvn clean package            # builder le jar
mvn test                     # lancer les tests
java -jar target/app.jar     # exécuter le jar buildé

# Gradle (équivalents)
./gradlew bootRun
./gradlew build
./gradlew test

# Activer un profil
java -jar app.jar --spring.profiles.active=prod
```

```java
// Annotations essentielles à connaître par cœur
@SpringBootApplication
@RestController @RequestMapping @GetMapping @PostMapping @PutMapping @DeleteMapping
@PathVariable @RequestParam @RequestBody
@Service @Repository @Component @Autowired
@Entity @Id @GeneratedValue @OneToMany @ManyToOne
@Valid @NotBlank @NotNull @Min @Max
@Transactional
@RestControllerAdvice @ExceptionHandler
@SpringBootTest @WebMvcTest @MockBean
```

## 7. Exercices

Énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/).

- **Niveau 1** — [`exercices/niveau-1.md`](exercices/niveau-1.md) : premier contrôleur REST, configuration de base.
- **Niveau 2** — [`exercices/niveau-2.md`](exercices/niveau-2.md) : Spring Data JPA, validation, gestion d'exceptions.
- **Niveau 3** — [`exercices/niveau-3.md`](exercices/niveau-3.md) : architecture en couches complète, cas proche d'une situation professionnelle.

## 8. Mini-projet

**API de gestion de bibliothèque** : endpoints CRUD pour des livres et des auteurs (relation `@ManyToOne`), validation des entrées, DTOs séparés des entités, gestion centralisée des erreurs (`@RestControllerAdvice`), pagination sur la liste des livres, un test d'intégration `@WebMvcTest` par endpoint principal. Bonus : sécuriser un endpoint d'écriture avec Spring Security (auth basique ou JWT).

## Checklist

- [ ] Comprendre les fondamentaux (IoC, auto-configuration, structure d'un projet)
- [ ] Savoir créer un projet (Spring Initializr, Maven/Gradle)
- [ ] Maîtriser la syntaxe principale (annotations REST, DI, JPA)
- [ ] Comprendre les concepts importants (couches, validation, gestion d'exceptions)
- [ ] Savoir debugger (logs Spring, erreurs d'auto-configuration, `NoSuchBeanDefinitionException`)
- [ ] Connaître les bonnes pratiques (constructor injection, DTOs, pagination, éviter le N+1)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Security, WebFlux, performance JPA, Actuator)

## 10. Ressources

- [Documentation officielle Spring Boot](https://docs.spring.io/spring-boot/index.html)
- [Spring Framework Reference](https://docs.spring.io/spring-framework/reference/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Baeldung](https://www.baeldung.com/) — articles techniques de référence sur l'écosystème Spring
- [Spring Initializr](https://start.spring.io)

Pas de roadmap.sh dédiée à Spring Boot à ce jour ; voir [roadmap.sh/java](https://roadmap.sh/java) pour le socle Java.
