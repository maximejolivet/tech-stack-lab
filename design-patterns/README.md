# Design Patterns, SOLID & Clean Code

## 1. Introduction

Un **design pattern** est une solution éprouvée et nommée à un problème de conception récurrent. Ce n'est pas du code à copier-coller : c'est un **vocabulaire partagé** ("on met un Strategy ici") qui permet à une équipe de discuter d'architecture sans réinventer les mots à chaque fois. **SOLID** et le **Clean Code** sont les principes sous-jacents qui expliquent *pourquoi* ces patterns existent et *quand* ils sont justifiés.

**À quoi ça sert ?**
- Rendre le code plus facile à faire évoluer sans tout casser (ajouter une fonctionnalité sans modifier l'existant).
- Rendre le code testable (dépendances remplaçables, responsabilités isolées).
- Donner un langage commun à l'équipe pour parler d'architecture.

**Où ça se situe dans une architecture web ?** Transverse à toute la stack : ces principes s'appliquent aussi bien à du PHP orienté objet (Symfony, Laravel) qu'à du JavaScript/TypeScript (services, composants, state management). Les frameworks eux-mêmes sont construits dessus — comprendre l'Injection de Dépendances, c'est comprendre *pourquoi* Symfony et Spring Boot sont conçus comme ils le sont.

**Avantages** : code plus lisible et prévisible, changement localisé (un bug ou une évolution ne se propage pas partout), testabilité (mocker une dépendance devient trivial).
**Limites** : mal appliqués, patterns et principes deviennent de la **sur-ingénierie** — complexité ajoutée sans bénéfice réel, code plus dur à suivre qu'un simple `if`. Un pattern est un outil pour un problème donné, pas un objectif en soi.

> Ce dossier est **transverse** : les exemples alternent PHP et JavaScript/TypeScript selon ce qui illustre le mieux chaque notion. Les mêmes principes s'appliquent dans les deux langages.

## 2. Prérequis

- Programmation orientée objet de base (classes, interfaces, héritage) — voir `php/` et `javascript/`
- Avoir écrit du code dans un vrai projet (idéalement avoir déjà ressenti la douleur d'un code difficile à faire évoluer)

## 3. Rappel des bases 🟢

### Pourquoi ces principes existent

Le code est lu bien plus souvent qu'il n'est écrit. Sans discipline, une base de code grossit en devenant de plus en plus coûteuse à modifier : chaque ajout de fonctionnalité risque de casser autre chose ailleurs. SOLID et Clean Code sont des garde-fous empiriques (pas des lois mathématiques) qui limitent ce coût dans le temps.

### 01 - S : Single Responsibility Principle

**Explication** — Une classe ne doit avoir **qu'une seule raison de changer**. Pas "une seule méthode" : une seule *responsabilité métier*.

```php
// ❌ Avant : deux responsabilités mélangées (métier + persistance)
class Invoice {
    public function calculateTotal(): float { /* ... */ }
    public function saveToDatabase(): void { /* SQL direct ici */ }
}

// ✅ Après : responsabilités séparées
class Invoice {
    public function calculateTotal(): float { /* ... */ }
}

class InvoiceRepository {
    public function save(Invoice $invoice): void { /* SQL ici */ }
}
```

**Cas d'usage** : dès qu'une classe grossit et mélange calcul métier, accès aux données et formatage de sortie — signe qu'il faut la scinder.
**Erreur fréquente** : confondre SRP avec "une classe = une méthode" → sur-découpage artificiel qui éparpille une logique cohérente dans 10 fichiers.
**Bonne pratique** : se demander "pour quelle raison métier cette classe changerait-elle ?" — si la réponse a un "et", c'est probablement deux responsabilités.

### 02 - O : Open/Closed Principle

**Explication** — Une classe doit être **ouverte à l'extension, fermée à la modification**. On ajoute du comportement en ajoutant du code (nouvelle classe), pas en modifiant du code qui marche déjà.

```js
// ❌ Avant : chaque nouveau type de remise oblige à modifier calculateDiscount
function calculateDiscount(type, amount) {
  if (type === "vip") return amount * 0.2;
  if (type === "student") return amount * 0.1;
  // ajouter un type = modifier cette fonction (risque de régression)
}

// ✅ Après : extensible sans toucher au code existant
class VipDiscount { apply(amount) { return amount * 0.2; } }
class StudentDiscount { apply(amount) { return amount * 0.1; } }

function calculateDiscount(strategy, amount) {
  return strategy.apply(amount);
}
```

**Cas d'usage** : logique métier qui varie selon un type et qui s'enrichit régulièrement (moyens de paiement, règles de tarification, formats d'export).
**Erreur fréquente** : appliquer l'OCP par anticipation partout ("on pourrait avoir besoin d'étendre un jour") → complexité inutile pour un besoin hypothétique (voir YAGNI, section 5).
**Bonne pratique** : n'introduire cette abstraction que lorsque l'extension est déjà arrivée deux fois (règle empirique du "Rule of Three").

### 03 - L : Liskov Substitution Principle

**Explication** — Une sous-classe doit pouvoir remplacer sa classe parente **sans casser le comportement attendu** par le code appelant.

```php
// ❌ Violation classique : le carré "hérite" du rectangle mais casse le contrat
class Rectangle {
    protected int $width;
    protected int $height;
    public function setWidth(int $w): void { $this->width = $w; }
    public function setHeight(int $h): void { $this->height = $h; }
    public function area(): int { return $this->width * $this->height; }
}

class Square extends Rectangle {
    public function setWidth(int $w): void { $this->width = $this->height = $w; } // surprise !
    public function setHeight(int $h): void { $this->width = $this->height = $h; }
}
// Un code qui fait setWidth(5); setHeight(10); attend area() === 50, pas 100.
```

**Cas d'usage** : concevoir des hiérarchies de classes ou implémenter une interface — le test mental est "puis-je remplacer le parent par l'enfant sans surprise ?".
**Erreur fréquente** : hériter pour réutiliser du code sans respecter le contrat sémantique du parent (ex. lever une exception dans une méthode censée ne jamais échouer).
**Bonne pratique** : si une sous-classe doit vider ou contredire une méthode du parent, c'est un signal pour repenser la hiérarchie (souvent vers de la composition, voir section 5).

### 04 - I : Interface Segregation Principle

**Explication** — Préférer plusieurs interfaces petites et spécifiques à une seule grosse interface générale. Une classe ne doit pas être forcée d'implémenter des méthodes dont elle n'a pas besoin.

```ts
// ❌ Avant : interface trop large
interface Worker {
  work(): void;
  eat(): void;
}
class Robot implements Worker {
  work() { /* ... */ }
  eat() { throw new Error("un robot ne mange pas"); } // forcé d'implémenter pour rien
}

// ✅ Après : interfaces séparées
interface Workable { work(): void; }
interface Eatable { eat(): void; }
class Robot implements Workable { work() { /* ... */ } }
class Human implements Workable, Eatable { work() { /* ... */ } eat() { /* ... */ } }
```

**Cas d'usage** : conception d'interfaces/contrats partagés par plusieurs implémentations hétérogènes.
**Erreur fréquente** : une interface "fourre-tout" qui grossit au fil du temps et force des implémentations vides ou des exceptions "not implemented".
**Bonne pratique** : découper les interfaces par capacité (rôle), pas par type d'objet.

### 05 - D : Dependency Inversion Principle

**Explication** — Les modules de haut niveau (logique métier) ne doivent pas dépendre de modules de bas niveau (détails techniques), mais tous deux dépendre d'**abstractions**.

```php
// ❌ Avant : la classe métier dépend directement d'une implémentation concrète
class OrderService {
    private MySqlDatabase $db; // couplage fort à MySQL
    public function __construct() { $this->db = new MySqlDatabase(); }
}

// ✅ Après : dépend d'une abstraction, l'implémentation est injectée
interface OrderRepositoryInterface {
    public function save(Order $order): void;
}

class OrderService {
    public function __construct(private OrderRepositoryInterface $repository) {}
}
// MySqlOrderRepository implements OrderRepositoryInterface { ... }
// InMemoryOrderRepository implements OrderRepositoryInterface { ... } (pour les tests)
```

**Cas d'usage** : c'est la base de l'**Injection de Dépendances** (approfondie section 5) — permet de tester `OrderService` avec un faux repository sans base de données réelle.
**Erreur fréquente** : instancier ses dépendances avec `new` à l'intérieur d'une classe métier au lieu de les recevoir — rend la classe impossible à tester isolément et couplée à une techno précise.
**Bonne pratique** : dépendre d'interfaces/contrats dans le code métier, injecter les implémentations concrètes de l'extérieur (constructeur le plus souvent).

### 06 - Clean Code : les bases

**Nommage** — un nom doit répondre à "pourquoi ça existe, ce que ça fait, comment c'est utilisé" sans commentaire nécessaire :

```js
// ❌
const d = 5; // délai en jours

// ✅
const delayInDays = 5;
```

**Fonctions courtes, un seul niveau d'abstraction** — une fonction qui mélange une boucle bas niveau et un appel métier haut niveau est plus dure à lire qu'une fonction découpée en étapes nommées.

**Éviter la duplication (DRY)** — mais attention : deux bouts de code qui *se ressemblent aujourd'hui* pour des raisons métier différentes ne doivent pas être fusionnés prématurément (voir "DRY mal appliqué", section 5).

**Commentaires utiles vs inutiles** :

```js
// ❌ Commentaire qui répète le code (inutile, devient obsolète)
// incrémente le compteur de 1
counter++;

// ✅ Commentaire qui explique un "pourquoi" non évident
// Stripe exige un montant en centimes, pas en euros (cf. leur doc API)
const amountInCents = amount * 100;
```

**Erreur fréquente** : commenter *ce que* fait le code (le code déjà bien nommé le dit) plutôt que *pourquoi* une décision non évidente a été prise.
**Bonne pratique** : si le code a besoin d'un commentaire pour être compris, essayer d'abord de le rendre plus lisible (renommer, extraire une fonction) ; garder le commentaire pour l'information que le code ne peut pas porter seul (contrainte externe, workaround, décision d'architecture).

## 4. Concepts intermédiaires 🟡

### Patterns créationnels

**Factory** — centralise la création d'objets complexes ou variables, pour ne pas disperser des `new` conditionnels dans toute l'app :

```php
interface PaymentMethod { public function pay(float $amount): void; }
class CreditCardPayment implements PaymentMethod { public function pay(float $amount): void {} }
class PaypalPayment implements PaymentMethod { public function pay(float $amount): void {} }

class PaymentFactory {
    public static function create(string $type): PaymentMethod {
        return match ($type) {
            'card' => new CreditCardPayment(),
            'paypal' => new PaypalPayment(),
            default => throw new InvalidArgumentException("Type inconnu: $type"),
        };
    }
}
```

**Singleton** — garantit une instance unique globale. **À utiliser avec prudence** : introduit un état global caché, rend les tests difficiles (état partagé entre tests), et masque les dépendances réelles d'une classe. En PHP/JS moderne, un conteneur d'injection de dépendances (voir section 5) remplace avantageusement le Singleton pour la plupart des cas (ex. connexion DB partagée).

```js
// Anti-pattern à connaître, pas à reproduire systématiquement
class Config {
  static #instance;
  static getInstance() {
    if (!Config.#instance) Config.#instance = new Config();
    return Config.#instance;
  }
}
```

**Builder** — construit un objet complexe étape par étape, utile quand un constructeur aurait trop de paramètres optionnels :

```ts
class QueryBuilder {
  private conditions: string[] = [];
  where(condition: string): this { this.conditions.push(condition); return this; }
  build(): string { return `SELECT * FROM table WHERE ${this.conditions.join(" AND ")}`; }
}
const query = new QueryBuilder().where("age > 18").where("active = 1").build();
```

### Patterns structurels

**Adapter** — traduit une interface incompatible vers celle attendue, typiquement pour intégrer une librairie tierce sans la coupler au reste du code :

```php
interface NotificationSender { public function send(string $message): void; }

class LegacySmsGateway { public function sendSms(string $text, string $number): bool { /* API tierce */ } }

class SmsAdapter implements NotificationSender {
    public function __construct(private LegacySmsGateway $gateway, private string $number) {}
    public function send(string $message): void {
        $this->gateway->sendSms($message, $this->number);
    }
}
```

**Decorator** — ajoute un comportement à un objet dynamiquement, sans modifier sa classe ni créer une explosion de sous-classes :

```ts
interface Coffee { cost(): number; }
class SimpleCoffee implements Coffee { cost() { return 2; } }
class MilkDecorator implements Coffee {
  constructor(private coffee: Coffee) {}
  cost() { return this.coffee.cost() + 0.5; }
}
const order = new MilkDecorator(new SimpleCoffee()); // 2.5
```

**Facade** — expose une interface simple au-dessus d'un sous-système complexe (plusieurs classes/APIs), pour cacher la complexité au code appelant.

### Patterns comportementaux

**Strategy** — encapsule une famille d'algorithmes interchangeables. Exemple réel : plusieurs moyens de paiement, ou plusieurs algorithmes de tri/tarification :

```php
interface ShippingStrategy { public function cost(float $weight): float; }
class StandardShipping implements ShippingStrategy { public function cost(float $weight): float { return $weight * 0.5; } }
class ExpressShipping implements ShippingStrategy { public function cost(float $weight): float { return $weight * 1.2 + 5; } }

class Order {
    public function __construct(private ShippingStrategy $shipping) {}
    public function shippingCost(float $weight): float { return $this->shipping->cost($weight); }
}
```

**Observer** — un sujet notifie une liste d'observateurs quand son état change, sans les connaître à l'avance. Base des event emitters (Node.js `EventEmitter`, events DOM, event bus applicatif) :

```js
class EventBus {
  #listeners = new Map();
  on(event, callback) {
    if (!this.#listeners.has(event)) this.#listeners.set(event, []);
    this.#listeners.get(event).push(callback);
  }
  emit(event, payload) {
    (this.#listeners.get(event) ?? []).forEach(cb => cb(payload));
  }
}
const bus = new EventBus();
bus.on("order.created", (order) => sendConfirmationEmail(order));
bus.emit("order.created", { id: 42 });
```

**Command** — encapsule une action (et ses paramètres) dans un objet, permettant de la mettre en file d'attente, de la logger, ou de l'annuler (undo). Base des jobs asynchrones (queues) en backend.

**Erreur fréquente transverse** : choisir un pattern pour "faire propre" avant d'avoir le problème qu'il résout — voir la section suivante sur la sur-ingénierie.

## 5. Concepts avancés 🟠🔴

### Composition over inheritance

L'héritage crée un couplage fort et rigide (une sous-classe dépend de toute l'implémentation de son parent, y compris ses détails internes). La composition — assembler des objets qui exposent un comportement via interface, plutôt que d'hériter — est presque toujours plus flexible :

```ts
// ❌ Hiérarchie rigide : que faire d'un canard qui ne vole pas ?
class Bird { fly() { /* ... */ } }
class Duck extends Bird {} // et un pingouin ?

// ✅ Composition : le comportement est injecté, pas hérité
interface FlyBehavior { fly(): void; }
class CanFly implements FlyBehavior { fly() { /* ... */ } }
class CannotFly implements FlyBehavior { fly() { /* no-op */ } }

class Duck {
  constructor(private flyBehavior: FlyBehavior) {}
  performFly() { this.flyBehavior.fly(); }
}
```

Règle empirique : préférer "a un" (composition, injection) à "est un" (héritage) dès que la hiérarchie risque de devoir représenter des combinaisons de comportements plutôt qu'une simple spécialisation.

### Dependency Injection & Inversion of Control

L'**Inversion of Control (IoC)** est le principe général : le framework/conteneur contrôle le flux et fournit les dépendances, plutôt que le code applicatif ne les construise lui-même. La **Dependency Injection (DI)** en est la technique la plus courante : les dépendances sont *injectées* (constructeur, propriété) plutôt qu'instanciées à l'intérieur.

```php
// Symfony / Laravel résolvent automatiquement ce type de dépendance
// via leur conteneur de services (autowiring), à partir des type-hints du constructeur
class OrderController {
    public function __construct(
        private OrderRepositoryInterface $orders,
        private LoggerInterface $logger,
    ) {}
}
```

C'est directement l'application du **D de SOLID** (section 3) à l'échelle d'une application entière : le conteneur DI est ce qui permet de remplacer une implémentation par une autre (prod vs tests) sans toucher au code métier. Spring Boot (Java) fonctionne sur exactement le même principe (`@Autowired`).

### Repository pattern

Abstrait l'accès aux données derrière une interface métier, pour que la logique applicative ignore *comment* et *où* les données sont stockées (SQL, API externe, cache) :

```php
interface UserRepositoryInterface {
    public function findById(int $id): ?User;
    public function save(User $user): void;
}
// Implémentations interchangeables : Doctrine, requêtes SQL brutes, in-memory pour les tests
```

Bénéfice principal : testabilité (un `InMemoryUserRepository` pour les tests unitaires, sans base de données réelle) et découplage de l'ORM.

### CQRS (aperçu)

**Command Query Responsibility Segregation** : séparer les opérations de lecture (*queries*, qui ne modifient rien) des opérations d'écriture (*commands*, qui modifient l'état), parfois jusqu'à utiliser des modèles de données différents pour chacune. Utile dans des systèmes à forte charge de lecture ou avec des règles métier d'écriture complexes — **sur-ingénierie garantie** sur un CRUD classique. À connaître pour le vocabulaire, rarement à appliquer en dehors de systèmes à forte complexité métier ou de volumétrie.

### Quand NE PAS utiliser un pattern

C'est la compétence la plus avancée du sujet : savoir reconnaître la **sur-ingénierie**.

- **YAGNI** (*You Aren't Gonna Need It*) : ne pas construire une abstraction pour un besoin futur hypothétique — le futur besoin, une fois arrivé, sera probablement différent de ce qu'on avait anticipé.
- **Rule of Three** : n'extraire une abstraction générique qu'à la troisième occurrence d'un besoin similaire, pas à la première.
- **DRY mal appliqué** : fusionner deux bouts de code qui se ressemblent *aujourd'hui* mais représentent des règles métier différentes crée un couplage accidentel — le jour où l'une des deux règles change, on casse l'autre.
- Un pattern ajouté "parce que c'est la bonne pratique" sans problème concret à résoudre est un signal d'alerte en revue de code.

### Maintenabilité à l'échelle d'une équipe

Ces principes ne sont pas des règles absolues mais des heuristiques à appliquer avec jugement : leur valeur se mesure au coût de changement dans le temps, pas à leur présence. Une équipe qui comprend *pourquoi* un pattern est là (le problème qu'il résout) le fait évoluer correctement ; une équipe qui l'applique par cargo-culting le fige ou le contourne.

## 6. Commandes / syntaxe à connaître

Pas d'outil CLI dédié — ce dossier est conceptuel. Repères syntaxiques rapides :

```php
// PHP : déclarer un contrat
interface PaymentMethod { public function pay(float $amount): void; }
class CardPayment implements PaymentMethod { public function pay(float $amount): void {} }

// Injection par constructeur (property promotion PHP 8+)
class Service {
    public function __construct(private PaymentMethod $payment) {}
}
```

```ts
// TypeScript : déclarer un contrat
interface PaymentMethod { pay(amount: number): void; }
class CardPayment implements PaymentMethod { pay(amount: number) {} }

// Injection par constructeur
class Service {
  constructor(private payment: PaymentMethod) {}
}
```

## 7. Exercices

Énoncés dans `exercices/`, corrections séparées dans `solutions/` — à ne consulter qu'après avoir essayé.

- **Niveau 1 — Bases** : `exercices/niveau-1.md`
- **Niveau 2 — Intermédiaire** : `exercices/niveau-2.md`
- **Niveau 3 — Avancé** : `exercices/niveau-3.md`

## 8. Mini-projet

**Système de notification multi-canal** (au choix PHP ou TypeScript) :

- Une interface `NotificationChannel` (Strategy) avec au moins deux implémentations (`EmailChannel`, `SmsChannel`)
- Un `NotificationService` qui reçoit ses canaux par injection de dépendances (constructeur), respectant le D de SOLID
- Un `EventBus` (Observer) qui déclenche l'envoi de notifications sur un événement `user.registered`
- Un `Decorator` `LoggingChannel` qui enveloppe n'importe quel canal pour logger chaque envoi sans modifier les classes existantes
- Une suite de tests qui utilise une implémentation `InMemoryChannel` à la place des vrais canaux (démontre la testabilité apportée par la DI)

Livrable : un dossier de code minimal (pas besoin de framework), avec un `README.md` d'une dizaine de lignes expliquant quel pattern résout quel problème dans ce mini-projet.

## Checklist

- [ ] Comprendre les fondamentaux (les 5 principes SOLID, bases du Clean Code)
- [ ] Savoir créer un projet structuré selon ces principes
- [ ] Maîtriser la syntaxe principale (interfaces, injection par constructeur, en PHP et en JS/TS)
- [ ] Comprendre les concepts importants (patterns créationnels, structurels, comportementaux courants)
- [ ] Savoir identifier un code qui viole SOLID à la lecture
- [ ] Connaître les bonnes pratiques (composition over inheritance, DI/IoC)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Repository, CQRS, quand NE PAS appliquer un pattern)

## 10. Ressources

- [Refactoring.Guru — Design Patterns](https://refactoring.guru/design-patterns) — référence moderne, très visuelle, exemples multi-langages
- [Refactoring.Guru — SOLID Principles](https://refactoring.guru/design-patterns/what-is-a-pattern) — vue d'ensemble solide
- *Design Patterns: Elements of Reusable Object-Oriented Software* (Gang of Four, 1994) — l'ouvrage fondateur, référence historique pour le vocabulaire des 23 patterns classiques
- *Clean Code* — Robert C. Martin — référence sur le nommage, les fonctions, les commentaires
- [Symfony — Service Container & Autowiring](https://symfony.com/doc/current/service_container.html) — application concrète de DI/IoC en PHP
- [Martin Fowler — Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html) — article de référence sur IoC/DI
