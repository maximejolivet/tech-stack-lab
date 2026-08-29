# Exercices PostgreSQL — Niveau 3 (Avancé)

## Exercice 1 — Bloat et autovacuum

Une table `sessions` reçoit des milliers d'`UPDATE` par minute (mise à jour du timestamp de dernière activité) et grossit anormalement en espace disque malgré un nombre de lignes stable. Explique le mécanisme MVCC responsable de ce phénomène, et le rôle d'`autovacuum` pour le contenir.

## Exercice 2 — EXPLAIN ANALYZE

Explique la différence entre `EXPLAIN` seul et `EXPLAIN ANALYZE` sur une même requête, et pourquoi utiliser `EXPLAIN ANALYZE` sur une requête `DELETE`/`UPDATE` en production nécessite une précaution particulière.

## Exercice 3 — SERIALIZABLE

Deux transactions concurrentes lisent chacune le solde total d'un compte puis y ajoutent un montant selon une règle métier (ex. "n'autoriser le dépôt que si le solde total du client, tous comptes confondus, ne dépasse pas 10 000€"). Explique pourquoi `REPEATABLE READ` ne suffit pas à garantir cette règle sous forte concurrence, et comment `SERIALIZABLE` la garantit.

## Exercice 4 — Extension PostGIS (conception)

Une application doit trouver tous les magasins dans un rayon de 5 km autour d'un point GPS donné. Décris (sans nécessairement écrire tout le SQL PostGIS) quel type de colonne utiliser, quelle extension activer, et la forme générale de la requête de recherche par proximité.

## Exercice 5 — Réplication logique ciblée

Une entreprise veut répliquer uniquement la table `products` (pas toute la base) vers un système de reporting externe, potentiellement sur une version PostgreSQL différente. Explique en 2-3 phrases pourquoi la réplication logique est adaptée à ce cas plutôt que la réplication physique (streaming).
