# Symfony

## 1. Introduction

Symfony est un framework PHP full-stack, orienté architecture et composants réutilisables. Il fournit une structure, un système d'injection de dépendances, un ORM (via Doctrine), un moteur de templates (Twig) et un ensemble de "composants" utilisables indépendamment (même hors Symfony, ex. dans Laravel ou Drupal). Ce dossier suppose le PHP natif acquis (voir [`../php/`](../php/)) et se concentre sur ce que Symfony ajoute par-dessus.

**À quoi sert-il ?**
- Structurer une application PHP de taille moyenne à grande (API, back-office, e-commerce) avec des conventions claires.
- Éviter de réécrire l'infrastructure (routing, DI, sécurité, formulaires, validation) à chaque projet.
- Servir de socle à des projets plus lourds (API Platform, e-commerce Sylius, Drupal 9+ qui repose en partie sur ses composants).

**Où se situe-t-il dans une architecture web ?**
Côté serveur : reçoit une requête HTTP (via le `Kernel`), la fait passer par un pipeline d'event listeners, la route vers un contrôleur, qui orchestre services/Doctrine, et retourne une `Response` (HTML via Twig, ou JSON pour une API).

**Avantages**
- Architecture très structurée : convention over configuration modérée, tout est explicite et testable (DI par constructeur).
- Composants découplés et réutilisables (HttpFoundation, Console, Validator...) utilisés dans une bonne partie de l'écosystème PHP.
- Excellent pour les projets qui doivent durer et évoluer en équipe (typage fort, autowiring, tests facilités par la DI).

**Limites**
- Courbe d'apprentissage plus raide que Laravel : plus de concepts explicites à comprendre avant d'être productif (services, DI container, event dispatcher).
- Verbeux pour un petit projet/prototype — Laravel ou un micro-framework va souvent plus vite pour du MVP.
- Doctrine ORM a ses propres pièges de performance (N+1 queries) à apprendre en plus du SQL.

## 2. Prérequis

- PHP 8.2+ maîtrisé : POO (interfaces, classes abstraites), namespaces, autoload PSR-4, Composer — voir [`../php/`](../php/) si besoin.
- Notions de base de données relationnelles (SQL, jointures) — voir [`../mysql/`](../mysql/).
- Symfony CLI installée (`symfony check:requirements`), Composer à jour.

## 3. Rappel des bases 🟢

### 01 - Créer un projet et le lancer

**Explication** — Deux façons de démarrer : `symfony new` (CLI Symfony, inclut un serveur local) ou `composer create-project symfony/skeleton` (squelette minimal, sans dépendances web).

```bash
symfony new mon-projet --webapp   # projet complet (Twig, sécurité, forms...)
cd mon-projet
symfony server:start -d           # serveur local avec TLS auto-signé
```

**Cas d'usage** : `--webapp` pour une app avec vues (back-office, site) ; skeleton minimal pour une API pure.

**Bonne pratique** : utiliser la Symfony CLI en dev (détection de version PHP, TLS local, tail des logs) plutôt que `php -S` directement.

### 02 - Structure d'un projet

**Explication** — Convention de répertoires fixe :

```text
mon-projet/
├── bin/console          # point d'entrée CLI
├── config/               # config YAML (services, routes, packages)
├── public/               # document root (index.php uniquement exposé)
├── src/                  # code applicatif (Controller, Entity, Repository...)
├── templates/            # vues Twig
├── tests/                # tests PHPUnit
├── var/                  # cache, logs (généré, jamais versionné)
└── vendor/               # dépendances Composer (jamais versionné)
```

**Erreur fréquente** : pointer le vhost/serveur web sur la racine du projet au lieu de `public/` → expose `src/`, `.env`, `vendor/` publiquement. Le document root DOIT toujours être `public/`.

### 03 - Routing

**Explication** — Une route associe une URL à une méthode de contrôleur. Depuis PHP 8, on utilise les attributs natifs plutôt que les annotations en commentaire.

```php
// src/Controller/ArticleController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ArticleController
{
    #[Route('/articles/{id}', name: 'article_show', requirements: ['id' => '\d+'])]
    public function show(int $id): Response
    {
        return new Response("Article #{$id}");
    }
}
```

**Bonne pratique** : contraindre les paramètres de route avec `requirements` (ex. `\d+` pour un id numérique) pour éviter des routes ambiguës et des erreurs 500 sur des conversions de type invalides.

**Erreur fréquente** : oublier `name:` sur une route puis devoir la retrouver par son chemin en dur dans les templates au lieu d'utiliser `path('article_show', {id: 5})` — casse dès que l'URL change.

### 04 - Contrôleurs et Response

**Explication** — Un contrôleur est une classe/méthode qui reçoit la `Request` (implicitement via des arguments typés) et retourne une `Response`. Étendre `AbstractController` donne accès à des raccourcis (`render()`, `redirectToRoute()`, `json()`).

```php
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class ArticleController extends AbstractController
{
    #[Route('/articles', name: 'article_list')]
    public function list(): Response
    {
        $articles = ['PHP 8.3', 'Symfony 7'];

        return $this->render('article/list.html.twig', ['articles' => $articles]);
    }

    #[Route('/api/articles', name: 'api_article_list')]
    public function apiList(): Response
    {
        return $this->json(['data' => ['PHP 8.3', 'Symfony 7']]);
    }
}
```

**Cas d'usage** : `render()` pour une page HTML, `json()` pour une API.

### 05 - Twig, les bases

**Explication** — Moteur de templates avec échappement automatique des variables (protection XSS par défaut).

```twig
{% extends 'base.html.twig' %}

{% block body %}
  <ul>
    {% for article in articles %}
      <li>{{ article|upper }}</li>
    {% else %}
      <li>Aucun article</li>
    {% endfor %}
  </ul>
{% endblock %}
```

**Bonne pratique** : ne jamais désactiver l'échappement (`|raw`) sur du contenu utilisateur — c'est la porte ouverte au XSS. Réserver `|raw` au contenu de confiance (HTML généré côté serveur par un éditeur WYSIWYG validé).

**Erreur fréquente** : mettre de la logique métier complexe dans les templates Twig au lieu de la préparer dans le contrôleur/service — Twig doit rester une couche de présentation.

### 06 - Configuration

**Explication** — Configuration en YAML dans `config/packages/*.yaml`, variables d'environnement dans `.env` (valeurs par défaut versionnées) et `.env.local` (secrets locaux, non versionné).

```yaml
# config/packages/framework.yaml
framework:
    secret: '%env(APP_SECRET)%'
```

**Bonne pratique** : ne jamais committer `.env.local` ni de vrais secrets dans `.env` — `.env` contient des valeurs de démo/dev uniquement, versionnées comme documentation des variables attendues.

### 07 - Bundles

**Explication** — Un bundle est un plugin Symfony (ex. `DoctrineBundle`, `SecurityBundle`). La plupart du framework lui-même est livré sous forme de bundles activés dans `config/bundles.php`, généré automatiquement par Symfony Flex à l'installation d'un paquet.

**Bonne pratique** : ne pas créer de bundle "maison" pour organiser son propre code applicatif — depuis Symfony 4+, c'est un anti-pattern. Organiser `src/` par domaine métier (dossiers) suffit.

## 4. Concepts intermédiaires 🟡

### Service Container et Dependency Injection

**Explication** — Symfony instancie et connecte automatiquement les objets ("services") via un conteneur DI. L'**autowiring** résout les dépendances d'un constructeur par leur type.

```php
namespace App\Service;

class ArticlePublisher
{
    public function __construct(
        private readonly ArticleRepository $repository,
        private readonly MailerInterface $mailer,
    ) {}

    public function publish(Article $article): void
    {
        $article->setPublishedAt(new \DateTimeImmutable());
        $this->repository->save($article);
        $this->mailer->send(/* ... */);
    }
}
```

Injecté ensuite tel quel dans un contrôleur ou un autre service, sans configuration manuelle : Symfony résout le graphe de dépendances.

**Bonne pratique** : privilégier l'injection par constructeur (pas de service locator `$container->get()`), et typer sur des **interfaces** plutôt que des implémentations concrètes pour rester testable (mock facile).

**Erreur fréquente** : rendre un service `public` sans raison — par défaut les services sont `private` (accessibles uniquement par injection), ce qui est voulu pour éviter le couplage global.

### Doctrine ORM

**Explication** — Mappe des classes PHP à des tables SQL via des attributs.

```php
#[ORM\Entity(repositoryClass: ArticleRepository::class)]
class Article
{
    #[ORM\Id, ORM\GeneratedValue, ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private string $title;

    #[ORM\ManyToOne(targetEntity: Author::class)]
    private Author $author;
}
```

```bash
php bin/console make:migration        # génère une migration à partir du diff d'entités
php bin/console doctrine:migrations:migrate
```

**Erreur fréquente** : boucler sur une relation (`$article->getComments()`) dans une vue/liste → problème N+1 (une requête SQL par article). Corriger avec un `JOIN` explicite dans le repository (`leftJoin`/`addSelect`) ou un fetch eager ciblé.

**Bonne pratique** : garder les entités "riches" mais sans logique d'accès aux données — les requêtes complexes vivent dans le `Repository`, pas dans le contrôleur.

### Formulaires et validation

**Explication** — Les `FormType` décrivent un formulaire de façon déclarative, liés à un objet (souvent une entité), avec validation via des attributs `#[Assert\...]`.

```php
#[Assert\NotBlank]
#[Assert\Length(min: 5, max: 255)]
private string $title;
```

**Bonne pratique** : toujours valider côté serveur même si une validation HTML5/JS existe côté client (voir [`../html/`](../html/)) — le client ne doit jamais être la seule ligne de défense.

### Sécurité (authentification, firewalls, voters)

**Explication** — `config/packages/security.yaml` définit des "firewalls" (zones protégées) et des `access_control` (règles d'accès par URL). Les autorisations fines se font via des **Voters** (logique métier "cet utilisateur peut-il éditer CET article ?").

```php
class ArticleVoter extends Voter
{
    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();
        return $subject instanceof Article && $subject->getAuthor() === $user;
    }
}
```

Voir [`../security/`](../security/) pour les principes de sécurité applicative transverses (OWASP, gestion des mots de passe).

### Tests fonctionnels

**Explication** — `WebTestCase` fournit un client HTTP simulé qui boote le kernel Symfony sans vrai serveur.

```php
class ArticleControllerTest extends WebTestCase
{
    public function testListReturns200(): void
    {
        $client = static::createClient();
        $client->request('GET', '/articles');

        $this->assertResponseIsSuccessful();
    }
}
```

Voir [`../testing/`](../testing/) pour la méthodologie de test transverse.

## 5. Concepts avancés 🟠🔴

### Event Dispatcher

**Explication** — Symfony diffuse des événements (kernel.request, kernel.response, événements métier custom) auxquels on peut s'abrancher sans modifier le code émetteur — mécanisme central de découplage.

```php
class ArticlePublishedSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [ArticlePublishedEvent::class => 'onArticlePublished'];
    }

    public function onArticlePublished(ArticlePublishedEvent $event): void
    {
        // notifier, invalider un cache, etc.
    }
}
```

Voir [`../design-patterns/`](../design-patterns/) pour le pattern Observer sous-jacent.

### Messenger (bus de messages, asynchrone)

**Explication** — Découple l'émission d'une action (ex. "envoyer un email") de son exécution, potentiellement différée via une queue (RabbitMQ, Redis, Doctrine transport).

```php
$this->bus->dispatch(new SendWelcomeEmail($user->getId()));
```

Un `Handler` traite le message, en synchrone ou via `php bin/console messenger:consume async` en asynchrone — utile pour ne pas bloquer la requête HTTP sur un traitement lent (envoi mail, génération de PDF).

### Cache

**Explication** — Symfony expose un `CacheInterface` PSR-6/PSR-16 agnostique du backend (filesystem, Redis — voir [`../redis/`](../redis/)), plus le cache HTTP (Reverse proxy Symfony ou Varnish) piloté par les en-têtes `Cache-Control`/ESI.

**Bonne pratique** : invalider explicitement par tag (`cache.tags`) plutôt que par TTL court partout — un TTL trop court annule le bénéfice du cache, un TTL trop long sert des données périmées.

### Compilation du container et performance

**Explication** — En production, le container DI est **compilé** une fois (résolution des services figée en PHP généré) plutôt que reconstruit à chaque requête. `bin/console cache:warmup` précompile ce qui peut l'être.

**Bonne pratique** : toujours vérifier que `APP_ENV=prod` et `opcache.validate_timestamps=0` sont actifs en production — sans ça, PHP revérifie le hash de chaque fichier à chaque requête, coût non négligeable à l'échelle.

### Architecture hexagonale / DDD avec Symfony

**Explication** — Symfony n'impose pas de découpage en couches métier/infrastructure : rien n'empêche (et c'est recommandé sur un gros projet) d'isoler le domaine métier pur (`src/Domain/`) de l'infrastructure (`src/Infrastructure/Doctrine/...`) pour ne pas coupler la logique métier à Doctrine/Symfony. Les Voters, Event Subscribers et le Message Bus sont des points d'extension naturels pour respecter cette séparation. Voir [`../system-design/`](../system-design/) pour la théorie d'architecture.

### API Platform (aperçu)

**Explication** — Bundle qui génère une API REST/GraphQL complète (CRUD, pagination, filtres, doc OpenAPI) à partir des entités Doctrine annotées. Utile pour aller vite sur une API standard ; moins adapté si les règles métier des endpoints divergent fortement du CRUD basique. Voir [`../api/`](../api/) pour la conception d'API indépendante du framework.

## 6. Commandes / syntaxe à connaître

```bash
# Projet
symfony new mon-projet --webapp
symfony server:start -d
symfony server:log

# Composer / Flex
composer require symfony/orm-pack
composer require --dev symfony/maker-bundle

# Génération de code (MakerBundle)
php bin/console make:controller ArticleController
php bin/console make:entity Article
php bin/console make:migration
php bin/console make:form ArticleType
php bin/console make:voter ArticleVoter

# Base de données
php bin/console doctrine:database:create
php bin/console doctrine:migrations:migrate
php bin/console doctrine:fixtures:load

# Cache et debug
php bin/console cache:clear
php bin/console debug:router
php bin/console debug:container
php bin/console debug:autowiring

# Messenger
php bin/console messenger:consume async

# Tests
php bin/phpunit
```

## 7. Exercices

Les énoncés sont dans [`exercices/`](exercices/) (niveau 1 à 3), les corrections dans [`solutions/`](solutions/). Essaie sans regarder les solutions.

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**API de gestion de tâches (todo-list) avec authentification**

Construire une petite API Symfony :
- Entité `Task` (titre, description, statut, date d'échéance) persistée via Doctrine/migrations.
- Entité `User` avec authentification (login par formulaire ou API token), chaque tâche appartient à un utilisateur.
- Endpoints JSON : liste des tâches de l'utilisateur connecté, création, mise à jour du statut, suppression — protégés par un Voter (un utilisateur ne peut modifier que ses propres tâches).
- Validation des entrées (`#[Assert\...]`), tests fonctionnels sur au moins 2 endpoints.
- Bonus : dispatcher un événement `TaskCompletedEvent` à la complétion d'une tâche et l'écouter dans un Subscriber qui logue l'action.

## Checklist

- [ ] Comprendre les fondamentaux (routing, contrôleurs, Twig)
- [ ] Savoir créer un projet Symfony et le lancer en local
- [ ] Maîtriser la syntaxe principale (attributs de route, DI par constructeur)
- [ ] Comprendre les concepts importants (service container, Doctrine, sécurité)
- [ ] Savoir debugger (`debug:router`, `debug:container`, profiler Symfony)
- [ ] Connaître les bonnes pratiques (services privés, validation serveur, N+1 queries)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Event Dispatcher, Messenger, architecture hexagonale)

## 10. Ressources

- [Documentation officielle Symfony](https://symfony.com/doc/current/index.html)
- [Symfony Best Practices](https://symfony.com/doc/current/best_practices.html)
- [Documentation Doctrine ORM](https://www.doctrine-project.org/projects/doctrine-orm/en/current/index.html)
- [SymfonyCasts](https://symfonycasts.com/) — formations vidéo officielles, très à jour
- Il n'existe pas de roadmap.sh dédiée à Symfony ; le [roadmap.sh PHP](https://roadmap.sh/php) couvre le langage, pas le framework.
