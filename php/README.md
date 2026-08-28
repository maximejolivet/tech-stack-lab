# PHP

## 1. Introduction

PHP est un langage de script côté serveur, historiquement conçu pour le web, exécuté à chaque requête HTTP (modèle classique) ou en mode long-running (FrankenPHP, Swoole, RoadRunner). Ce dossier couvre le langage **natif** (syntaxe, POO, fonctionnalités PHP 8.x) — les frameworks ([`../symfony/`](../symfony/), [`../laravel/`](../laravel/)) et les runtimes modernes ([`../frankenphp/`](../frankenphp/)) ont chacun leur propre dossier.

**À quoi sert-il ?**
- Générer du HTML dynamique côté serveur (rendu classique ou SSR).
- Exposer des API (REST/GraphQL) consommées par un front JS ou une app mobile.
- Traiter des données (formulaires, fichiers, jobs asynchrones via des workers).

**Où se situe-t-il dans une architecture web ?**
Côté serveur exclusivement : il reçoit une requête HTTP, interagit avec une base de données/cache/services externes, et renvoie une réponse (HTML, JSON...). Il ne s'exécute jamais dans le navigateur.

**Avantages**
- Déploiement simple et très répandu (hébergement mutualisé à cloud natif).
- Écosystème mature (Composer, PSR, Symfony/Laravel), excellent pour le CRUD et les CMS.
- Typage progressif depuis PHP 7/8 : on peut être strict sans perdre la flexibilité du langage.

**Limites**
- Modèle "share-nothing" par défaut (process tué à chaque requête en PHP-FPM classique) : pas d'état partagé en mémoire entre requêtes sans outil dédié (Redis, runtime persistant).
- Historique chargé de fonctions incohérentes (`strpos` vs `str_contains`, ordre des arguments variable) — en grande partie assaini mais à connaître.
- Moins naturel que Node.js pour du temps réel (WebSockets) sans outillage additionnel (Swoole, Ratchet).

## 2. Prérequis

- Bases de programmation générale (variables, fonctions, boucles) — voir [`../javascript/`](../javascript/) si besoin de rafraîchir ces notions transverses.
- PHP 8.2+ installé en local (`php -v`), Composer installé (`composer --version`).
- Un terminal et un éditeur avec extension PHP (Intelephense, PHPStorm...) pour l'autocomplétion et la détection d'erreurs de type.

## 3. Rappel des bases 🟢

### 01 - Syntaxe et balises

**Explication** — Le code PHP s'ouvre avec `<?php` et se ferme avec `?>` (omis en pratique dans un fichier 100% PHP, pour éviter d'émettre un caractère blanc parasite après la balise fermante).

```php
<?php

declare(strict_types=1); // active le typage strict pour ce fichier

echo "Hello, world!\n";
```

**Bonne pratique** : toujours démarrer un fichier PHP pur par `declare(strict_types=1);` — sans elle, PHP convertit silencieusement les types passés aux fonctions typées (`"5"` accepté pour un paramètre `int`), ce qui masque des bugs.

**Erreur fréquente** : laisser un `?>` en fin de fichier suivi d'un espace/newline → "headers already sent" si le fichier ne doit produire que du code (fichiers de classes, config). Ne jamais fermer la balise PHP dans ce cas.

### 02 - Variables et types scalaires

**Explication** — Variables préfixées par `$`, typage dynamique par défaut. Types scalaires : `int`, `float`, `string`, `bool`. Types composés : `array`, `object`. Types spéciaux : `null`, `callable`, `iterable`, `mixed`.

```php
$name = "Max";           // string
$age = 30;                 // int
$price = 19.99;            // float
$isActive = true;          // bool

var_dump($name);  // string(3) "Max"
gettype($age);      // "integer"
```

**Erreur fréquente** : confondre `==` (égalité "loose", avec conversion de type) et `===` (égalité stricte). PHP a des cas de coercition célèbres (`0 == "abc"` est `false` depuis PHP 8, mais `"1" == "01"` reste `true`).

```php
var_dump(0 == "abc");   // false (corrigé en PHP 8, était true en PHP 7)
var_dump("1" == "01");  // true  (comparaison numérique de deux chaînes numériques)
var_dump("1" === "01"); // false (types identiques requis, ici même type mais valeurs différentes en chaîne)
```

**Bonne pratique** : `===`/`!==` par défaut, `declare(strict_types=1)` pour forcer le typage des paramètres/retours de fonction.

### 03 - Constantes

**Explication** — `const` (au niveau classe/global, résolu à la compilation) et `define()` (résolu à l'exécution, plus rare aujourd'hui).

```php
const MAX_RETRIES = 3;

class Config
{
    public const DEFAULT_TIMEOUT = 30;
}

echo Config::DEFAULT_TIMEOUT;
```

**Bonne pratique** : préférer `const` à `define()` (performance légèrement meilleure, autocomplétion IDE, cohérence avec les constantes de classe).

### 04 - Opérateurs

**Explication** — Arithmétiques, comparaison, logiques, et opérateurs PHP spécifiques utiles : `??` (null coalescing), `??=`, `<=>` (spaceship, pour `usort`).

```php
$user = null;
$name = $user?->name ?? "Anonyme"; // nullsafe + null coalescing combinés

$count = 0;
$count ??= 10; // assigne 10 seulement si $count est null (ici $count = 0, donc inchangé)

usort($items, fn($a, $b) => $a->price <=> $b->price); // tri croissant par prix
```

**Erreur fréquente** : utiliser `isset($array['key']) ? $array['key'] : 'default'` alors que `$array['key'] ?? 'default'` fait la même chose en plus court et gère aussi les index inexistants sans warning.

### 05 - Structures de contrôle

**Explication** — `if/elseif/else`, `switch`, et `match` (PHP 8, expression stricte — voir section avancée).

```php
if ($age >= 18) {
    $status = "majeur";
} elseif ($age >= 13) {
    $status = "ado";
} else {
    $status = "enfant";
}

switch ($role) {
    case "admin":
        grantFullAccess();
        break;
    case "editor":
        grantEditAccess();
        break;
    default:
        grantReadAccess();
}
```

**Erreur fréquente** : oublier un `break` dans un `switch` (fall-through implicite, contrairement à `match` qui n'a pas ce piège).

**Bonne pratique** : dès que la logique est un simple mapping valeur → résultat, préférer `match` (voir avancé) à `switch`.

### 06 - Boucles

**Explication** — `for`, `while`, `do...while`, `foreach` (la plus utilisée pour parcourir des tableaux).

```php
$fruits = ["pomme", "poire", "kiwi"];

foreach ($fruits as $index => $fruit) {
    echo "$index: $fruit\n";
}

foreach ($fruits as &$fruit) {  // par référence, pour MODIFIER le tableau
    $fruit = strtoupper($fruit);
}
unset($fruit); // impératif après un foreach par référence
```

**Erreur fréquente** : oublier `unset($fruit)` après un `foreach` par référence — la variable `$fruit` reste liée au dernier élément et peut corrompre un `foreach` suivant qui réutilise le même nom de variable.

**Bonne pratique** : n'utiliser le `foreach` par référence (`&$fruit`) que quand c'est nécessaire ; sinon préférer `array_map` pour transformer un tableau sans mutation.

### 07 - Fonctions

**Explication** — Typage des paramètres et du retour, valeurs par défaut, variadic (`...$args`), passage par référence (`&$var`, à éviter sauf cas justifié).

```php
function greet(string $name, string $greeting = "Bonjour"): string
{
    return "$greeting, $name !";
}

function sum(int ...$numbers): int
{
    return array_sum($numbers);
}

sum(1, 2, 3); // 6 — les arguments variadiques sont collectés dans un tableau
```

**Erreur fréquente** : typer les paramètres sans `declare(strict_types=1)` en pensant être protégé — sans cette directive, PHP convertit silencieusement `"3"` en `int` pour un paramètre `int $x`.

**Bonne pratique** : toujours typer paramètres ET retour (`: string`, `: void`...) — c'est de la documentation exécutable et ça permet à l'IDE/l'analyse statique (PHPStan/Psalm) de détecter des bugs avant l'exécution.

### 08 - Tableaux

**Explication** — Un seul type `array` pour les tableaux indexés et associatifs (structure hybride ordered map). Fonctions essentielles : `array_map`, `array_filter`, `array_reduce`, `in_array`, `array_key_exists`.

```php
$prices = [10, 25, 8, 42];
$product = ["name" => "Clavier", "price" => 49.99];

$withTax = array_map(fn($p) => $p * 1.2, $prices);
$expensive = array_filter($prices, fn($p) => $p > 20);
$total = array_reduce($prices, fn($carry, $p) => $carry + $p, 0);

array_key_exists("price", $product); // true — vérifie la clé, même si la valeur est null
isset($product["price"]);              // true — false si la valeur est null !
```

**Erreur fréquente** : `array_filter` préserve les clés d'origine (le tableau résultant a des "trous" d'index) — un `json_encode` derrière peut alors produire un objet `{}` au lieu d'un tableau `[]`. Utiliser `array_values()` pour ré-indexer si besoin.

**Bonne pratique** : `isset()` pour vérifier rapidement une clé non-null (plus rapide), `array_key_exists()` quand une valeur `null` doit être considérée comme "existante".

### 09 - Chaînes de caractères

**Explication** — Guillemets simples (littéral, pas d'interpolation) vs doubles (interpolation de variables), heredoc/nowdoc pour les blocs multi-lignes.

```php
$name = "Max";
echo 'Bonjour, $name';   // Bonjour, $name (littéral)
echo "Bonjour, $name";   // Bonjour, Max (interpolé)
echo "Bonjour, {$name}!"; // accolades pour désambiguïser (obligatoire si accès à une propriété/méthode)

str_contains($str, "needle"); // PHP 8+, remplace strpos($str, "needle") !== false
str_starts_with($str, "pre");
str_ends_with($str, "fix");
```

**Erreur fréquente** : `strpos($str, "needle") == false` — si `"needle"` est trouvé en position `0`, `strpos` retourne `0`, qui est falsy, donc ce test est incorrect. Toujours comparer avec `=== false`, ou utiliser `str_contains()` (PHP 8+) qui évite le problème.

**Bonne pratique** : privilégier les guillemets simples pour du texte sans variable (légèrement plus rapide, intention claire), et les fonctions `str_contains`/`str_starts_with`/`str_ends_with` plutôt que `strpos` pour la lisibilité.

### 10 - Superglobales

**Explication** — Tableaux disponibles dans tout le code sans `global` : `$_GET`, `$_POST`, `$_SERVER`, `$_SESSION`, `$_COOKIE`, `$_FILES`, `$_ENV`.

```php
$page = $_GET['page'] ?? 1;
$method = $_SERVER['REQUEST_METHOD'];
```

**Erreur fréquente** : faire confiance aveuglément aux superglobales (données 100% contrôlées par le client) — injection possible si utilisées directement dans une requête SQL ou affichées sans échappement. Voir [`../security/`](../security/) pour l'approfondissement.

**Bonne pratique** : dans un projet moderne (Symfony/Laravel), ne jamais accéder aux superglobales directement — passer par l'objet Request du framework, qui centralise validation et échappement.

### 11 - Inclusion de fichiers

**Explication** — `require`/`require_once` (erreur fatale si le fichier est absent) vs `include`/`include_once` (simple warning). En pratique moderne, l'autoload Composer (PSR-4, voir intermédiaire) remplace la plupart des `require` manuels pour les classes.

```php
require_once __DIR__ . '/config.php'; // chemin absolu, robuste au répertoire courant
```

**Bonne pratique** : toujours utiliser `__DIR__` plutôt qu'un chemin relatif (qui dépend du répertoire depuis lequel le script est lancé) ; réserver `require`/`include` manuels aux fichiers de config/bootstrap, laisser l'autoload gérer les classes.

## 4. Concepts intermédiaires 🟡

- **POO — classes et visibilité** : `public`/`protected`/`private`, propriétés typées, constructeur, méthodes statiques.

```php
class User
{
    public function __construct(
        private readonly string $name, // property promotion (PHP 8) : déclare + assigne en un temps
        private int $age,
    ) {}

    public function getName(): string
    {
        return $this->name;
    }
}
```

- **Héritage, interfaces, classes abstraites, traits** : `extends` (héritage simple), `implements` (contrat, multiple), classe `abstract` (ne peut être instanciée, peut définir des méthodes abstraites à implémenter), `trait` (réutilisation de code horizontale, pas d'héritage réel — utile pour éviter la duplication sans hiérarchie forcée).
- **Namespaces et autoload PSR-4** : un namespace par dossier logique, mappé sur le système de fichiers ; Composer génère l'autoload à partir de `composer.json` (`"autoload": {"psr-4": {"App\\": "src/"}}`).
- **Composer** : `composer.json` (dépendances, autoload, scripts), `composer install` (respecte `composer.lock`, reproductible), `composer update` (remonte les versions selon les contraintes semver). Ne jamais commiter `vendor/`, toujours commiter `composer.lock` pour une application (pas pour une librairie).
- **Gestion des erreurs — exceptions** : hiérarchie `Throwable` → `Error` (erreurs internes PHP, ex. `TypeError`) / `Exception` (erreurs applicatives). Créer des exceptions métier spécifiques plutôt que des `Exception` génériques.

```php
final class UserNotFoundException extends \RuntimeException
{
    public function __construct(int $id)
    {
        parent::__construct("Utilisateur $id introuvable");
    }
}

try {
    $user = $repository->find($id) ?? throw new UserNotFoundException($id); // throw en expression, PHP 8
} catch (UserNotFoundException $e) {
    // traitement spécifique
} finally {
    $connection->close();
}
```

- **JSON** : `json_encode`/`json_decode` — par défaut `json_decode` retourne des `stdClass`, passer `true` en 2ᵉ argument pour obtenir des tableaux associatifs.
- **Sessions et cookies** : `session_start()`, `$_SESSION`, `setcookie()` — en pratique, un framework gère ça via un objet Session dédié (sécurité : `httponly`, `secure`, `samesite`).
- **Union types et nullable types** (PHP 8) : `function find(int|string $id): ?User` — un paramètre peut accepter plusieurs types explicitement listés, `?Type` est un raccourci pour `Type|null`.

## 5. Concepts avancés 🟠🔴

- **Enums** (PHP 8.1) : type dédié pour un ensemble fini de valeurs, remplace avantageusement les constantes de classe pour ce cas.

```php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Shipped = 'shipped';
    case Delivered = 'delivered';

    public function label(): string
    {
        return match($this) {
            self::Pending => 'En attente',
            self::Shipped => 'Expédiée',
            self::Delivered => 'Livrée',
        };
    }
}
```

- **`match` (PHP 8)** : comparaison stricte (`===`), pas de fall-through, doit couvrir tous les cas (sinon `UnhandledMatchError`) — plus sûr que `switch` pour du mapping valeur → résultat.
- **Named arguments, nullsafe operator (`?->`), first-class callable syntax** :

```php
createUser(name: "Max", age: 30); // ordre libre, lisibilité accrue avec beaucoup de paramètres optionnels

$city = $user?->address?->city; // court-circuite en null dès qu'un maillon est null, pas d'exception

$mapper = strtoupper(...); // first-class callable (PHP 8.1), remplace 'strtoupper' en string ou Closure::fromCallable
array_map($mapper, $names);
```

- **Attributes (PHP 8)** : métadonnées natives (`#[Route('/users')]`), remplacent les annotations en commentaire (utilisées massivement par Symfony/Doctrine modernes).
- **Générateurs (`yield`)** : produisent une séquence de valeurs à la demande, sans charger toute la collection en mémoire — essentiel pour traiter de gros volumes (lecture de fichier ligne à ligne, pagination de résultats DB).

```php
function readLines(string $path): \Generator
{
    $handle = fopen($path, 'r');
    while (($line = fgets($handle)) !== false) {
        yield trim($line); // suspend l'exécution, rend la main à l'appelant
    }
    fclose($handle);
}

foreach (readLines('big-file.csv') as $line) {
    // traite une ligne à la fois, mémoire constante quelle que soit la taille du fichier
}
```

- **Closures avancées** : `use (&$var)` pour capturer par référence, `Closure::bind`/`bindTo` pour changer le `$this` d'une closure.
- **Standards PSR** : PSR-1/PSR-12 (style de code — automatisé via PHP-CS-Fixer), PSR-4 (autoload), PSR-7 (HTTP messages), PSR-3 (logging) — connaître leur existence évite de réinventer des interfaces déjà standardisées dans l'écosystème.
- **Performance** : OPcache (met en cache le bytecode compilé, gain majeur en prod, à activer systématiquement), JIT (PHP 8, gains surtout sur du calcul pur, marginal sur une app web classique I/O-bound), profiling avec Xdebug ou Blackfire pour identifier les goulots réels avant d'optimiser à l'aveugle.
- **Fibers** (PHP 8.1, aperçu) : primitive bas niveau pour la concurrence coopérative, base de runtimes async modernes (Swoole, ReactPHP, ou du code Symfony async) — rarement manipulée directement, mais utile pour comprendre ce qui rend FrankenPHP/Swoole possibles.
- **Sécurité au niveau langage** : requêtes préparées PDO (jamais de concaténation de variables dans du SQL), `htmlspecialchars()` pour tout affichage de donnée utilisateur en HTML — approfondi dans [`../security/`](../security/), pas dupliqué ici.
- **Design orienté interfaces/composition** : dépendre d'interfaces plutôt que de classes concrètes (permet substitution/tests), composition (traits, injection de dépendances) plutôt qu'héritage profond — voir [`../design-patterns/`](../design-patterns/) pour les principes SOLID appliqués.

## 6. Commandes / syntaxe à connaître

```bash
php -v                      # version installée
php -S localhost:8000       # serveur de dev intégré (jamais en prod)
php script.php               # exécuter un script
php -l file.php              # lint syntaxique sans exécuter

composer init                # créer un composer.json
composer require vendor/pkg  # ajouter une dépendance
composer install             # installer selon composer.lock (reproductible)
composer update               # remonter les versions selon composer.json
composer dump-autoload        # régénérer l'autoload (après ajout manuel de classes)
```

```php
// Syntaxe essentielle à avoir sous les doigts
declare(strict_types=1);
$x ??= $default;
$obj?->method();
match ($status) { 'a' => 1, 'b' => 2, default => 0 };
try { /* ... */ } catch (SomeException $e) { /* ... */ } finally { /* ... */ }
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Gestionnaire de tâches en CLI, orienté objet**

Construire une petite application PHP en ligne de commande (pas de framework) qui doit :
- Modéliser une tâche avec une classe `Task` (propriétés typées, `readonly` où pertinent, un enum `TaskStatus`).
- Persister les tâches dans un fichier JSON (lecture/écriture via `json_encode`/`json_decode`).
- Exposer des commandes simples (`add`, `list`, `done <id>`, `delete <id>`) lues depuis `$argv`.
- Utiliser des exceptions métier custom (`TaskNotFoundException`) proprement catchées avec un message utilisateur clair.
- Utiliser `match` pour dispatcher la commande demandée vers la bonne méthode.

Objectif : mobiliser POO, typage strict, enums, match, gestion d'erreurs et manipulation de fichiers dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux (types, structures de contrôle, tableaux, fonctions)
- [ ] Savoir créer un projet PHP avec Composer (composer.json, autoload PSR-4)
- [ ] Maîtriser la syntaxe principale (POO, exceptions, closures)
- [ ] Comprendre les concepts importants (namespaces, interfaces/traits, typage progressif)
- [ ] Savoir debugger (Xdebug, `var_dump`, logs)
- [ ] Connaître les bonnes pratiques (strict_types, PSR, requêtes préparées)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (enums, générateurs, fibers, OPcache)

## 10. Ressources

- [Documentation officielle PHP](https://www.php.net/manual/fr/) — référence complète et à jour, toujours vérifier la version documentée.
- [PHP-FIG — PSR standards](https://www.php-fig.org/psr/) — standards d'interopérabilité (autoload, style, HTTP messages...).
- [PHP RFC — 8.x features](https://wiki.php.net/rfc) — pour comprendre le *pourquoi* des nouvelles fonctionnalités (enums, readonly, fibers...).
- [Composer documentation](https://getcomposer.org/doc/) — gestion des dépendances.
- [roadmap.sh — PHP](https://roadmap.sh/php) — vue d'ensemble du parcours d'apprentissage.
