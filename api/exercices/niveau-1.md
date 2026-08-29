# Exercices API Design — Niveau 1 (Bases)

## Exercice 1 — Nommer des routes RESTful

Une application de gestion de tâches a besoin des opérations suivantes sur une ressource `task` : lister, créer, voir le détail, modifier, supprimer, marquer comme terminée. Écris les routes RESTful correspondantes (verbe + chemin), y compris pour "marquer comme terminée" en respectant la sémantique REST (indice : c'est une modification partielle d'état, pas une nouvelle ressource).

## Exercice 2 — Corriger un mauvais usage de GET

Voici une route existante : `GET /orders/42/cancel`. Explique pourquoi c'est problématique et propose la route corrigée (verbe + chemin).

## Exercice 3 — Choisir le bon code de statut

Pour chacun des cas suivants, indique le code de statut HTTP approprié : (a) création réussie d'une ressource, (b) suppression réussie sans corps de réponse, (c) tentative d'accès à une ressource par un utilisateur non connecté, (d) validation de formulaire échouée, (e) ressource demandée introuvable.

## Exercice 4 — Structure d'erreur

Écris une réponse JSON d'erreur `422` pour un formulaire d'inscription où le champ `email` est déjà utilisé et le champ `password` fait moins de 8 caractères.

## Exercice 5 — Pagination simple

Écris la structure JSON d'une réponse paginée par offset pour `GET /products?page=2&per_page=10`, avec un total de 47 produits.
