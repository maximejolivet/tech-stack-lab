# Niveau 1 — Bases

## Exercice 1 — Routes et contrôleur

Crée une route `GET /hello/{name}` qui appelle un contrôleur `GreetingController@show` renvoyant `"Bonjour, {name} !"` en texte brut. Utilise route model binding... non, ici `{name}` est une simple string, pas un model — juste un paramètre de route classique.

## Exercice 2 — Vue Blade

Crée une vue `resources/views/greeting.blade.php` qui affiche une liste de prénoms passée par le contrôleur (`@foreach`), avec un `@if` qui affiche "Aucun prénom" si la liste est vide.

## Exercice 3 — Migration et model

Crée une migration pour une table `books` (`title` string, `pages` integer, `read` boolean par défaut `false`) et le model `Book` correspondant. Vérifie avec `php artisan tinker` que tu peux créer un livre via `Book::create([...])`.

## Exercice 4 — Config

Explique (en commentaire, pas besoin d'exécuter) pourquoi `env('APP_NAME')` dans un contrôleur est une mauvaise pratique, et quelle est l'alternative correcte.
