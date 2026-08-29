# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

EXPLAIN SELECT * FROM orders WHERE customer_id = 42;
-- vérifier que "type" passe de "ALL" à "ref" et que "key" indique idx_orders_customer_id
```

## Exercice 2

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 50 WHERE id = 1;
-- Si le solde de id=1 devient négatif ici, faire ROLLBACK; au lieu de continuer :
-- ROLLBACK;

UPDATE accounts SET balance = balance + 50 WHERE id = 2;

COMMIT;
```

## Exercice 3

```sql
SELECT customer_id, COUNT(*) AS nb_commandes
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

`HAVING` est nécessaire car le filtre porte sur `COUNT(*)`, une valeur **agrégée** calculée après le `GROUP BY` — `WHERE` s'applique avant l'agrégation, sur les lignes brutes, et ne peut donc pas référencer une fonction d'agrégat.

## Exercice 4

```sql
ALTER TABLE orders
    ADD CONSTRAINT fk_orders_customer
    FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE RESTRICT;
```

`ON DELETE RESTRICT` empêche de supprimer un client tant qu'il existe encore des commandes qui le référencent — la suppression échoue avec une erreur d'intégrité référentielle plutôt que de laisser des commandes orphelines.

## Exercice 5

Cette requête concatène directement une valeur utilisateur (`$_GET['email']`) dans la chaîne SQL : un attaquant peut injecter du SQL arbitraire via ce paramètre (ex. `' OR '1'='1`), contournant l'authentification ou exfiltrant des données — c'est une injection SQL classique. En PDO, on utilise une requête préparée avec paramètre lié :

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $_GET['email']]);
```

La valeur est alors transmise séparément de la requête, jamais interprétée comme du SQL.
