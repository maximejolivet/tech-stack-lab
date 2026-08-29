# Exercices Drupal — Niveau 3 (Avancé)

## Exercice 1 — Cache Tags

Le contrôleur JSON de l'annuaire d'événements (route `/evenements/a-venir`) doit rester correctement mis en cache mais s'invalider dès qu'un événement est créé, modifié ou supprimé. Explique quel Cache Tag utiliser dans le tableau de rendu retourné, et pourquoi.

## Exercice 2 — Endpoint JSON custom

Écris un contrôleur qui retourne, en `JsonResponse`, la liste des événements à venir (titre, date, lien) — équivalent Drupal de l'endpoint REST custom vu côté WordPress.

## Exercice 3 — Event Subscriber

Écris un `EventSubscriber` qui écoute l'événement `KernelEvents::REQUEST` et journalise (`\Drupal::logger()`) chaque requête vers une route commençant par `/evenements`. Explique en une phrase la différence entre cette approche et un `hook_*` procédural classique.

## Exercice 4 — Plugin Migrate (conception)

Décris (sans forcément écrire tout le YAML) la structure d'une migration Migrate API qui importe des événements depuis un fichier CSV (`title,date,location`) vers des nodes de type `event` : quelles sont les trois familles de plugins impliquées (source, process, destination) et le rôle de chacune ?

## Exercice 5 — BigPipe

Un bloc affichant "Bonjour {utilisateur connecté}" empêche une page par ailleurs entièrement cacheable d'être mise en cache pour tous les visiteurs. Explique en 2-3 phrases comment BigPipe résout ce problème, et quelle propriété de cache (`#lazy_builder` ou cache context) rend un élément de rendu éligible à ce traitement.
