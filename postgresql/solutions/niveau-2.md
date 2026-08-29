# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```sql
SELECT
    id,
    customer_id,
    order_date,
    total,
    SUM(total) OVER (PARTITION BY customer_id ORDER BY order_date) AS cumul_client
FROM orders
ORDER BY customer_id, order_date;
```

## Exercice 2

```sql
WITH totals AS (
    SELECT customer_id, SUM(total) AS total_depense
    FROM orders
    GROUP BY customer_id
)
SELECT * FROM totals WHERE total_depense > 500;
```

## Exercice 3

```sql
INSERT INTO products (sku, name, price) VALUES ('SKU-001', 'Clavier', 45.00)
ON CONFLICT (sku) DO UPDATE SET price = EXCLUDED.price;
```

## Exercice 4

```sql
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);
```

## Exercice 5

```sql
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id, 0 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT c.id, c.name, c.parent_id, ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY depth, name;
```
