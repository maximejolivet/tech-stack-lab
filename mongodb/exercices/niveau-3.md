# Exercices MongoDB — Niveau 3 (Avancé)

## Exercice 1 — Choix d'une clé de sharding

Une collection `events` (plusieurs milliards de documents) enregistre des événements applicatifs avec un champ `createdAt` (timestamp d'écriture) et un champ `tenantId` (identifiant du client SaaS propriétaire de l'événement). Explique pourquoi choisir `createdAt` seul comme clé de sharding créerait un "hot shard", et propose une clé de sharding plus adaptée.

## Exercice 2 — Replica set et lecture après écriture

Une application écrit un nouveau document sur le primaire d'un replica set, puis redirige immédiatement l'utilisateur vers une page qui lit ce document avec une préférence de lecture `secondaryPreferred`. Explique pourquoi cette page peut afficher "document introuvable", et propose une solution (indice : compare avec l'exercice équivalent déjà vu côté réplication MySQL dans [`../../mysql/exercices/niveau-3.md`](../../mysql/exercices/niveau-3.md)).

## Exercice 3 — Index composé pour une agrégation

Un pipeline d'agrégation fréquent filtre sur `{ status: "completed" }` (`$match`) puis trie par `createdAt` décroissant (`$sort`). Écris l'index composé qui sert au mieux cette combinaison filtre + tri, et explique en une phrase pourquoi l'ordre des champs dans l'index a de l'importance.

## Exercice 4 — Lecture d'un plan d'exécution

Voici un extrait simplifié du résultat de `.explain()` sur une requête `find` :

```
stage: "COLLSCAN"
nReturned: 42
totalDocsExamined: 3000000
```

Explique ce que signifie `COLLSCAN` sur une collection de 3 millions de documents pour ne retourner que 42 résultats, pourquoi c'est problématique, et quelle action corrective envisager.

## Exercice 5 — Quand ne pas utiliser MongoDB

Un système de comptabilité doit garantir qu'un virement entre deux comptes, impliquant potentiellement une dizaine de tables liées (comptes, écritures, journaux d'audit, contraintes réglementaires), reste strictement cohérent à tout instant, avec des contraintes d'intégrité référentielle vérifiées par la base elle-même. Argumente en 4-5 lignes pourquoi [`../../mysql/`](../../mysql/) ou [`../../postgresql/`](../../postgresql/) restent un meilleur choix que MongoDB pour ce cas précis, malgré la flexibilité de MongoDB ailleurs dans le même système d'information.
