# Exercices PostgreSQL — Niveau 2 (Intermédiaire)

## Exercice 1 — Window function

Étant donné `orders(id, customer_id, order_date, total)`, écris une requête qui affiche chaque commande avec le cumul des montants du même client jusqu'à cette commande incluse (triée par date).

## Exercice 2 — CTE simple

Réécris cette requête imbriquée en utilisant une CTE (`WITH`) pour améliorer sa lisibilité :

```sql
SELECT * FROM (
    SELECT customer_id, SUM(total) AS total_depense
    FROM orders
    GROUP BY customer_id
) AS totals
WHERE total_depense > 500;
```

## Exercice 3 — ON CONFLICT (upsert)

Écris un `INSERT` dans `products(sku, name, price)` (où `sku` est unique) qui met à jour `price` si le `sku` existe déjà, sans erreur de contrainte.

## Exercice 4 — Index GIN sur JSONB

Une requête filtrant sur `metadata->>'color' = 'red'` d'une table `products` avec 500 000 lignes est lente. Écris la commande qui crée l'index approprié pour l'accélérer.

## Exercice 5 — CTE récursive

Étant donné `categories(id, name, parent_id)`, écris une CTE récursive qui retourne l'arborescence complète à partir de la racine (`parent_id IS NULL`).
