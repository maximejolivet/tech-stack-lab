# Niveau 3 — Avancé

## Exercice 1 — Refactor "fat controller"

Voici un contrôleur qui fait tout dans une seule méthode `store()` (validation inline, création du livre, envoi d'un email de notification à l'auteur, log). Refactore-le : extrait la logique dans une classe `Action` ou `Service` dédiée, garde le contrôleur fin (validation via Form Request + appel à l'action + réponse). Justifie ton découpage en commentaire.

## Exercice 2 — Job asynchrone

Transforme l'envoi d'email de l'exercice 1 en Job dispatché sur une queue (`ShouldQueue`). Explique dans quel cas ça change concrètement le comportement perçu par l'utilisateur, et ce qu'il faut mettre en place pour que ce Job soit réellement traité (worker).

## Exercice 3 — API Resource et versioning

Crée une `BookResource` qui expose `id`, `title`, `author_name` (calculé depuis la relation, pas stocké tel quel en base) et cache `pages` du JSON de sortie. Explique comment tu gérerais une v2 de cette API qui doit exposer `pages` en plus, sans casser les clients existants de la v1.

## Exercice 4 — Diagnostic de performance

On te signale qu'une page qui liste 50 auteurs avec leurs livres met 3 secondes à charger. Décris ta démarche de diagnostic (outils, requêtes à examiner) et les 2-3 causes les plus probables à vérifier en premier, avec la correction associée.
