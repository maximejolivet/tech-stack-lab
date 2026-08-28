# Laravel

## 1. Introduction

Laravel est un framework PHP full-stack orienté productivité et "developer experience" : il fournit prêt-à-l'emploi le routing, l'ORM (Eloquent), le templating (Blade), l'authentification, les jobs asynchrones, le cache, etc. Ce dossier suppose le PHP natif acquis (voir [`../php/`](../php/)) et se concentre sur ce que Laravel ajoute par-dessus.

**À quoi sert-il ?**
- Construire rapidement des applications web CRUD, des API REST, des back-offices.
- Éviter de réécrire les briques transverses (auth, validation, ORM, queues) déjà standardisées et testées.
- Prototyper vite tout en gardant une trajectoire vers une architecture propre si le projet grossit.

**Où se situe-t-il dans une architecture web ?**
Côté serveur, en couche application : il reçoit la requête HTTP (via le routeur), orchestre la logique métier (contrôleurs/services), interagit avec la base de données (Eloquent) et renvoie une réponse (vue Blade, JSON, redirection).

**Avantages**
- Productivité élevée : beaucoup de "batteries incluses" (auth, queues, cache, notifications, scheduling).
- Documentation officielle excellente et écosystème riche (Forge, Vapor, Nova, Sanctum...).
- Eloquent est très ergonomique pour prototyper vite.

**Limites**
- "Magie" (façades, conventions implicites) qui peut masquer ce qui se passe réellement — utile de comprendre le Service Container sous-jacent.
- Eloquent facilite les pièges de performance (N+1) si on ne surveille pas les requêtes générées.
- Moins structurant que Symfony par défaut : la rigueur d'architecture dépend davantage de la discipline de l'équipe.

## 2. Prérequis

- PHP natif solide : POO, namespaces, Composer, gestion d'exceptions (voir [`../php/`](../php/)).
- Notions de base en SQL (voir [`../mysql/`](../mysql/)) pour comprendre ce qu'Eloquent génère réellement.
- Composer et PHP 8.2+ installés (`composer --version`, `php -v`).

## 3. Rappel des bases 🟢

### 01 - Installation et structure d'un projet

**Explication** — Un projet Laravel se crée via `laravel new` ou `composer create-project laravel/laravel`. La structure est conventionnelle : `app/` (code applicatif), `routes/` (définition des routes), `resources/` (vues, assets), `config/`, `database/` (migrations, seeders), `public/` (point d'entrée `index.php`).

```bash
laravel new mon-projet
cd mon-projet
php artisan serve   # serveur de dev sur localhost:8000
```

**Bonne pratique** : ne jamais modifier le contenu de `public/` en dehors des assets buildés — c'est le seul dossier exposé publiquement par le serveur web.

### 02 - Routing

**Explication** — Les routes web (avec session/CSRF) sont définies dans `routes/web.php`, les routes API (stateless) dans `routes/api.php`.

```php
// routes/web.php
use App\Http\Controllers\TaskController;

Route::get('/tasks', [TaskController::class, 'index']);
Route::post('/tasks', [TaskController::class, 'store']);
Route::get('/tasks/{task}', [TaskController::class, 'show']); // route model binding
```

**Cas d'usage** : `{task}` est automatiquement résolu en instance `Task` (route model binding) si le paramètre du contrôleur est typé `Task $task` — évite un `Task::findOrFail($id)` manuel.

**Erreur fréquente** : définir une route dynamique (`/tasks/{id}`) avant une route statique (`/tasks/create`) — Laravel matche dans l'ordre déclaré, donc `/tasks/create` tomberait dans `{id}`. Toujours déclarer les routes statiques avant les routes à paramètre.

### 03 - Contrôleurs

**Explication** — Classes qui reçoivent la `Request` et renvoient une `Response` (vue, JSON, redirection). Générés via Artisan.

```bash
php artisan make:controller TaskController --resource
```

```php
class TaskController extends Controller
{
    public function index(): View
    {
        return view('tasks.index', ['tasks' => Task::all()]);
    }
}
```

**Bonne pratique** : garder les contrôleurs fins (validation + appel à un service/action + réponse) — la logique métier va dans des classes dédiées (Services, Actions), pas dans le contrôleur (voir "fat controller" en avancé).

### 04 - Blade (templating)

**Explication** — Moteur de templates de Laravel, compilé en PHP pur (donc performant). Échappe automatiquement les variables affichées avec `{{ }}`.

```blade
{{-- resources/views/tasks/index.blade.php --}}
@extends('layouts.app')

@section('content')
    <ul>
        @foreach ($tasks as $task)
            <li>{{ $task->title }} — @if($task->done) ✅ @else ⏳ @endif</li>
        @endforeach
    </ul>
@endsection
```

**Erreur fréquente** : utiliser `{!! $variable !!}` (non échappé) pour afficher une donnée utilisateur — faille XSS directe. Réserver `{!! !!}` au HTML de confiance généré côté serveur (ex. contenu d'un éditeur WYSIWYG déjà purifié).

**Bonne pratique** : `{{ }}` par défaut systématiquement ; utiliser `@extends`/`@section`/`@yield` pour factoriser un layout commun plutôt que dupliquer le HTML.

### 05 - Configuration (.env)

**Explication** — Les paramètres d'environnement (DB, mail, clés API) vivent dans `.env` (jamais commité) et sont lus via `config/*.php` puis exposés par le helper `config()`.

```php
// config/database.php lit DB_HOST, DB_DATABASE... depuis .env
config('database.default'); // jamais env() directement en dehors des fichiers config/
```

**Erreur fréquente** : appeler `env('DB_HOST')` directement dans le code applicatif (contrôleurs, services) — une fois la config mise en cache (`php artisan config:cache` en prod), `env()` retourne `null` en dehors des fichiers `config/`. Toujours passer par `config()`.

### 06 - Migrations de base

**Explication** — Les migrations versionnent le schéma de base de données en code PHP, exécutables et réversibles.

```php
Schema::create('tasks', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->boolean('done')->default(false);
    $table->timestamps();
});
```

```bash
php artisan make:migration create_tasks_table
php artisan migrate
```

**Bonne pratique** : une migration = un changement atomique de schéma, jamais modifiée après avoir été jouée en production (créer une nouvelle migration pour corriger).

## 4. Concepts intermédiaires 🟡

- **Eloquent ORM — models et relations** : chaque table a un `Model` correspondant (convention de nommage singulier/PascalCase → table plurielle/snake_case).

```php
class Task extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}

class User extends Model
{
    public function tasks(): HasMany
    {
        return $this->hasMany(Task::class);
    }
}
```

- **Le piège N+1** : accéder à une relation dans une boucle déclenche une requête par itération.

```php
// ❌ N+1 : 1 requête pour les users + N requêtes pour chaque ->tasks
foreach (User::all() as $user) {
    echo $user->tasks->count();
}

// ✅ Eager loading : 2 requêtes au total
foreach (User::with('tasks')->get() as $user) {
    echo $user->tasks->count();
}
```

**Bonne pratique** : activer `Model::preventLazyLoading()` en environnement de dev/test pour détecter les N+1 dès le développement plutôt qu'en production.

- **Validation via Form Requests** : extraire la validation du contrôleur dans une classe dédiée.

```php
class StoreTaskRequest extends FormRequest
{
    public function rules(): array
    {
        return ['title' => 'required|string|max:255'];
    }
}

public function store(StoreTaskRequest $request): RedirectResponse
{
    Task::create($request->validated()); // déjà validé, prêt à l'emploi
    return redirect()->route('tasks.index');
}
```

- **Middlewares** : code exécuté avant/après une requête (auth, CORS, throttle). S'enregistrent par route ou groupe de routes.
- **Service Container et injection de dépendances** : Laravel résout automatiquement les dépendances typées dans les constructeurs de contrôleurs/jobs — comprendre ce mécanisme évite de traiter les "façades" (`Auth::`, `Cache::`) comme de la magie pure.
- **Authentification** : Breeze/Fortify pour un starter kit classique, Sanctum pour l'auth API par token — ne pas réimplémenter un système d'auth à la main.
- **Tests** : PHPUnit ou Pest, factories Eloquent pour générer des données de test, `RefreshDatabase` trait pour une base propre à chaque test.

## 5. Concepts avancés 🟠🔴

- **Jobs et Queues** : déporter un traitement lent (envoi d'email, génération de PDF) hors du cycle requête/réponse.

```php
class SendWelcomeEmail implements ShouldQueue
{
    use Queueable;

    public function __construct(private User $user) {}

    public function handle(): void
    {
        Mail::to($this->user)->send(new WelcomeMail($this->user));
    }
}

SendWelcomeEmail::dispatch($user); // exécuté par un worker, pas dans la requête HTTP
```

**Bonne pratique** : toute opération non essentielle à la réponse immédiate (email, notification, traitement de fichier) devrait passer par une queue plutôt que de ralentir la requête HTTP.

- **Events et Listeners** : découpler des effets de bord (ex. "UserRegistered" déclenche l'envoi d'email + la création d'un profil, sans coupler ces deux actions dans le contrôleur).
- **Cache et sessions** : driver Redis en production (voir [`../redis/`](../redis/)) plutôt que le driver fichier, pour la performance et le partage entre plusieurs serveurs applicatifs.
- **API Resources** : classes de transformation pour contrôler précisément la forme JSON exposée par une API, indépendamment de la structure interne du Model.

```php
class TaskResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return ['id' => $this->id, 'title' => $this->title, 'done' => (bool) $this->done];
    }
}
```

- **Éviter le "fat controller" et le "fat model"** : extraire la logique métier dans des classes Action/Service dédiées plutôt que de tout empiler dans le contrôleur ou le Model — voir [`../design-patterns/`](../design-patterns/) (Repository, composition, SOLID).
- **Performance en production** : `php artisan config:cache`, `route:cache`, `view:cache` (évite de re-parser les fichiers PHP à chaque requête), OPcache activé, file d'attente supervisée par Horizon (si Redis) pour monitorer les jobs.
- **Déploiement** : gérer les migrations en zero-downtime (colonnes ajoutées en nullable puis backfill), variables d'environnement séparées par environnement, `php artisan optimize` en CI/CD.

## 6. Commandes / syntaxe à connaître

```bash
laravel new mon-projet              # nouveau projet
php artisan serve                    # serveur de dev
php artisan make:controller Name     # générer un contrôleur
php artisan make:model Name -m       # générer un model + sa migration
php artisan make:migration create_x  # générer une migration seule
php artisan migrate                  # jouer les migrations
php artisan migrate:rollback         # annuler la dernière migration
php artisan db:seed                  # jouer les seeders
php artisan tinker                   # REPL interactif sur l'app
php artisan queue:work                # démarrer un worker de queue
php artisan config:cache              # mettre en cache la config (prod)
php artisan route:list                # lister toutes les routes
php artisan test                      # lancer les tests
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**API de gestion de tâches (To-Do API)**

Construire une API REST Laravel avec :
- Un model `Task` (migration, factory) lié à un `User` (`belongsTo`/`hasMany`).
- Des routes API CRUD (`GET/POST/PUT/DELETE /api/tasks`) protégées par Sanctum.
- Une `StoreTaskRequest`/`UpdateTaskRequest` pour la validation.
- Une `TaskResource` pour contrôler le format JSON de sortie.
- Un test Pest/PHPUnit qui vérifie qu'un utilisateur ne peut pas voir/modifier les tâches d'un autre utilisateur.
- Un job `SendTaskReminder` dispatché en queue quand une tâche approche de son échéance.

Objectif : mobiliser routing API, Eloquent + relations, validation, Resources, auth par token et queues dans un exercice réalisable en une session.

## Checklist

- [ ] Comprendre les fondamentaux (routing, contrôleurs, Blade, migrations)
- [ ] Savoir créer un projet Laravel (`laravel new`, structure du projet)
- [ ] Maîtriser la syntaxe principale (Eloquent, Form Requests, middlewares)
- [ ] Comprendre les concepts importants (Service Container, eager loading, auth)
- [ ] Savoir debugger (`tinker`, logs, requêtes générées par Eloquent)
- [ ] Connaître les bonnes pratiques (éviter N+1, fat controller, cache config en prod)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Jobs/Queues, Events, API Resources, déploiement)

## 10. Ressources

- [Documentation officielle Laravel](https://laravel.com/docs) — référence complète, toujours vérifier la version documentée.
- [Laracasts](https://laracasts.com/) — formation vidéo de référence dans l'écosystème Laravel.
- [Eloquent — documentation](https://laravel.com/docs/eloquent) — relations, requêtes, eager loading.
- Il n'existe pas de roadmap.sh dédiée à Laravel à ce jour ; voir [roadmap.sh — PHP](https://roadmap.sh/php) pour le contexte langage plus large.
