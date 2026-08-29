# Exercices MySQL — Niveau 3 (Avancé)

## Exercice 1 — Lecture du plan EXPLAIN

Voici un extrait simplifié de résultat `EXPLAIN` :

```
type: ALL   rows: 1200000   Extra: Using where
```

Explique ce que signifie `type: ALL` sur une table de 1,2 million de lignes, pourquoi c'est problématique, et quelle action corrective envisager.

## Exercice 2 — Deadlock

Deux transactions concurrentes verrouillent les lignes `id=1` puis `id=2` dans un ordre inversé l'une par rapport à l'autre, provoquant un deadlock détecté par MySQL. Explique en 2-3 phrases ce qui se passe côté MySQL quand un deadlock est détecté, et propose une modification du code applicatif qui réduit le risque qu'il se reproduise.

## Exercice 3 — Niveau d'isolation

Explique la différence entre `READ COMMITTED` et `REPEATABLE READ` (niveau par défaut de MySQL/InnoDB), à l'aide d'un exemple concret de lecture répétée dans une même transaction.

## Exercice 4 — Réplication et lecture juste après écriture

Une application écrit un nouvel utilisateur sur le serveur primaire, puis redirige immédiatement l'utilisateur vers une page qui lit son profil depuis une réplique en lecture seule. Explique pourquoi cette page peut afficher une erreur "utilisateur introuvable", et propose une solution.

## Exercice 5 — Dénormalisation justifiée

Une table `orders` normalisée nécessite une jointure avec `order_items` et `products` à chaque affichage de la liste des commandes, ce qui devient coûteux à grande échelle. Propose une dénormalisation ciblée (quelle donnée dupliquer, où) qui réduit ce coût, et explique en une phrase le compromis qu'elle introduit.
