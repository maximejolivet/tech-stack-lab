# Niveau 2 — Intermédiaire

## Exercice 1 — Relations et N+1

Modélise `Author` (`hasMany` `Book`) et `Book` (`belongsTo` `Author`). Écris une route qui liste tous les auteurs avec leur nombre de livres, en évitant volontairement le N+1 (utilise `with()` et explique pourquoi ta requête est optimale).

## Exercice 2 — Form Request

Crée une `StoreBookRequest` qui valide : `title` obligatoire (max 255), `pages` entier positif, `author_id` doit exister dans la table `authors`. Utilise-la dans un contrôleur `store()`.

## Exercice 3 — Middleware custom

Écris un middleware `EnsureBookLimit` qui bloque la création d'un livre (redirection avec message d'erreur) si l'utilisateur authentifié a déjà 10 livres ou plus. Applique-le uniquement sur la route `POST /books`.

## Exercice 4 — Tests

Écris un test (PHPUnit ou Pest, au choix) qui vérifie qu'un utilisateur non authentifié reçoit un 401/redirection sur `POST /books`, et qu'un utilisateur authentifié peut créer un livre avec des données valides.
