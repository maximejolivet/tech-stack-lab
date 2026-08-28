# Spring Boot — Exercices niveau 1 (Bases)

## Exercice 1 — Premier contrôleur

Crée un projet Spring Boot (via [start.spring.io](https://start.spring.io), dépendance "Spring Web") avec un `GreetingController` exposant :

- `GET /api/hello` → retourne `{"message": "Hello, World!"}`
- `GET /api/hello/{name}` → retourne `{"message": "Hello, {name}!"}` en utilisant `@PathVariable`

## Exercice 2 — Query params

Ajoute un endpoint `GET /api/greet?name=Max&lang=fr` qui retourne un message différent selon `lang` (`fr` → "Bonjour", `en` → "Hello", autre → 400 Bad Request). Utilise `@RequestParam`.

## Exercice 3 — Configuration

Change le port par défaut de l'application (8080 → 9090) via `application.properties`, puis via `application.yml`. Explique en une phrase la différence entre les deux formats.

## Exercice 4 — POST et corps de requête

Crée un endpoint `POST /api/echo` qui reçoit un JSON `{"text": "..."}` et le retourne tel quel, avec un code de statut `201 Created`.
