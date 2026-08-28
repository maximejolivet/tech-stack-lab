# Spring Boot — Exercices niveau 3 (Avancé)

## Exercice 1 — Architecture en couches complète

Construis une API `Order` (commande) avec une vraie séparation Controller / Service / Repository :
- Le `Controller` ne contient aucune logique métier (délègue tout au `Service`).
- Le `Service` gère les règles métier (ex : une commande ne peut pas être créée sans au moins une ligne d'article) et est annoté `@Transactional` sur les opérations d'écriture.
- Le `Repository` ne contient que de l'accès aux données.

Justifie en quelques lignes pourquoi cette séparation compte, avec un exemple concret d'anti-pattern ("fat controller") que tu évites.

## Exercice 2 — Pagination et performance

Le endpoint `GET /api/orders` doit être paginé (`Pageable`, `PageRequest`) plutôt que de retourner toute la table. Mesure (ou explique théoriquement) l'impact d'un `findAll()` sans pagination sur une table de plusieurs millions de lignes, et comment `Pageable` évite ce problème.

## Exercice 3 — Sécuriser un endpoint

Ajoute Spring Security au projet. Protège les endpoints d'écriture (`POST`/`PUT`/`DELETE`) pour qu'ils nécessitent une authentification, en laissant les `GET` publics. Documente la différence entre authentification (qui es-tu) et autorisation (as-tu le droit) dans ton implémentation.

## Exercice 4 — Cas proche du réel : diagnostiquer un N+1

On te donne (mentalement, ou en le codant) un endpoint qui liste des commandes avec leurs lignes d'articles, et qui déclenche visiblement plusieurs dizaines de requêtes SQL pour une liste de 10 commandes. Identifie la cause probable, propose et implémente une correction (`JOIN FETCH` ou `@EntityGraph`), puis explique comment tu aurais pu détecter ce problème plus tôt (logs SQL, `spring.jpa.show-sql=true`, outil de profiling).
