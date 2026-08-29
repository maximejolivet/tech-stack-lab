# Exercices MySQL — Niveau 2 (Intermédiaire)

## Exercice 1 — Index

Une requête `SELECT * FROM orders WHERE customer_id = 42` est lente sur une table de 2 millions de lignes. Écris la commande qui crée l'index approprié, et la commande `EXPLAIN` qui permettrait de vérifier qu'il est bien utilisé.

## Exercice 2 — Transaction

Écris une transaction SQL qui transfère 50€ du compte `id=1` vers le compte `id=2` dans une table `accounts(id, balance)`, avec `ROLLBACK` explicite en commentaire si le solde du compte source deviendrait négatif.

## Exercice 3 — HAVING vs WHERE

Écris une requête qui liste les clients ayant passé plus de 5 commandes, avec le nombre exact de commandes. Explique en une phrase pourquoi `HAVING` est nécessaire ici plutôt que `WHERE`.

## Exercice 4 — Clé étrangère et intégrité

Étant donné une table `orders` avec une colonne `customer_id` sans contrainte de clé étrangère, écris l'`ALTER TABLE` qui ajoute cette contrainte vers `customers(id)`, avec un comportement `ON DELETE RESTRICT`. Explique en une phrase ce que ce comportement empêche.

## Exercice 5 — Requête préparée

Explique en 2-3 phrases pourquoi cette requête est dangereuse, et comment un développeur PHP utilisant PDO l'écrirait de façon sécurisée :

```sql
SELECT * FROM users WHERE email = '" . $_GET['email'] . "'
```
