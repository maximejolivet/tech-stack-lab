# Exercices API Design — Niveau 3 (Avancé)

## Exercice 1 — Résoudre un N+1 côté API

Un client appelle `GET /authors` (liste de 20 auteurs) puis, pour chacun, `GET /authors/{id}/books` afin d'afficher leurs livres — soit 21 requêtes au total. Propose deux solutions différentes pour réduire ce nombre de requêtes, une côté REST et une avec GraphQL.

## Exercice 2 — Idempotence sur un paiement

Un endpoint `POST /payments` peut être appelé deux fois par accident si le client retente après un timeout réseau (sans savoir si la première requête a abouti). Explique comment une clé d'idempotence (`Idempotency-Key`) résout ce problème côté serveur.

## Exercice 3 — Webhook sécurisé

Une API tierce de paiement notifie ton backend via un webhook `POST /webhooks/payment-status` à chaque changement de statut. Explique pourquoi il ne faut pas faire confiance au corps de la requête tel quel, et comment vérifier son authenticité.

## Exercice 4 — Circuit breaker

Ton API appelle un service tiers de géolocalisation qui est actuellement en panne (100% d'erreurs 500). Explique le comportement d'un circuit breaker dans cette situation, en distinguant ses trois états (fermé, ouvert, semi-ouvert).

## Exercice 5 — Cache HTTP conditionnel

Écris les headers de requête/réponse impliqués dans un cycle de cache conditionnel basé sur `ETag`, pour un client qui a déjà en cache la version précédente d'une ressource `GET /products/7`.
