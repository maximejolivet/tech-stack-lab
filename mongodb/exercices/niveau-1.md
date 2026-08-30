# Exercices MongoDB — Niveau 1 (Bases)

## Exercice 1 — Connexion et insertion

Connecte-toi avec `mongosh`, bascule sur une base `mon_site`, puis insère 3 documents dans une collection `products` (`name`, `price`, `category`).

## Exercice 2 — Requête avec opérateurs de comparaison

Écris une requête qui sélectionne les produits dont le prix est supérieur à 20, triés par prix décroissant. Écris une seconde requête qui sélectionne les produits dont la catégorie est `"informatique"` ou `"bureau"` (utilise `$in`).

## Exercice 3 — Mise à jour ciblée

Écris une requête `updateMany` qui augmente de 10% le prix de tous les produits de la catégorie `"électronique"` (utilise `$mul` ou `$inc` selon ce qui te semble le plus adapté). Explique en une phrase ce qui se passerait si tu oubliais l'opérateur de mise à jour (`$set`/`$mul`/...) et passais directement `{ price: ... }`.

## Exercice 4 — Suppression avec condition

Écris une requête `deleteMany` qui supprime tous les produits dont le prix est inférieur à 10. Explique en une phrase le risque d'appeler `deleteMany({})`.

## Exercice 5 — Index

Une requête `db.products.find({ category: "informatique" })` est fréquente sur une collection de plusieurs millions de documents. Écris la commande qui crée l'index approprié, et la commande qui liste les index existants sur la collection.
