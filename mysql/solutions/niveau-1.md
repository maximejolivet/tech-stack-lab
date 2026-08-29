# Solutions — Niveau 1 (Bases)

## Exercice 1

```sql
CREATE TABLE products (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Exercice 2

```sql
INSERT INTO products (name, price) VALUES
    ('Clavier', 45.00),
    ('Souris', 15.00),
    ('Écran', 199.00);

SELECT * FROM products WHERE price > 20 ORDER BY price DESC;
```

## Exercice 3

```sql
UPDATE products SET price = price * 1.10 WHERE category = 'électronique';
```

Sans `WHERE`, l'`UPDATE` s'appliquerait à **toutes** les lignes de la table, augmentant le prix de tous les produits quelle que soit leur catégorie.

## Exercice 4

```sql
SELECT products.name, SUM(order_items.quantity) AS total_quantite
FROM products
JOIN order_items ON order_items.product_id = products.id
GROUP BY products.id, products.name;
```

## Exercice 5

```sql
SELECT customers.name, COUNT(orders.id) AS nb_commandes
FROM customers
LEFT JOIN orders ON orders.customer_id = customers.id
GROUP BY customers.id, customers.name;
```

Un `INNER JOIN` ne conserverait que les clients ayant **au moins une** commande correspondante — les clients sans commande disparaîtraient du résultat au lieu d'afficher 0.
