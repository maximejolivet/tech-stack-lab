# Exercices MySQL — Niveau 1 (Bases)

## Exercice 1 — Création de table

Écris le `CREATE TABLE` d'une table `products` avec `id` (clé primaire auto-incrémentée), `name` (texte, obligatoire), `price` (décimal, obligatoire), `created_at` (timestamp, valeur par défaut à maintenant).

## Exercice 2 — Insertions et sélection

Insère 3 produits dans la table `products`, puis écris une requête qui sélectionne les produits dont le prix est supérieur à 20, triés par prix décroissant.

## Exercice 3 — Mise à jour ciblée

Écris une requête `UPDATE` qui augmente de 10% le prix de tous les produits de la catégorie `'électronique'` (suppose une colonne `category`). Explique en une phrase le risque si la clause `WHERE` est oubliée.

## Exercice 4 — Jointure simple

Étant donné `products(id, name)` et `order_items(id, product_id, quantity)`, écris une requête qui liste le nom de chaque produit avec la quantité totale commandée (utilise `SUM` et `GROUP BY`).

## Exercice 5 — LEFT JOIN vs INNER JOIN

Étant donné `customers(id, name)` et `orders(id, customer_id)`, écris une requête qui liste **tous** les clients avec le nombre de commandes qu'ils ont passées (0 pour ceux qui n'en ont aucune). Explique pourquoi un `INNER JOIN` ne conviendrait pas ici.
