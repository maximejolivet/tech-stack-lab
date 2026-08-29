# Exercices PostgreSQL — Niveau 1 (Bases)

## Exercice 1 — Création de table avec IDENTITY

Écris le `CREATE TABLE` d'une table `articles` avec `id` (clé primaire `GENERATED ALWAYS AS IDENTITY`), `title` (texte, obligatoire), `tags` (tableau de texte).

## Exercice 2 — RETURNING

Écris un `INSERT` dans la table `articles` qui retourne directement l'`id` généré et non la ligne complète.

## Exercice 3 — Piège de casse

Explique pourquoi cette séquence de commandes échoue à la deuxième ligne, et corrige-la :

```sql
CREATE TABLE "Articles" (id INTEGER);
SELECT * FROM Articles;
```

## Exercice 4 — JSONB de base

Étant donné une table `products(id, metadata JSONB)` où `metadata` contient `{"color": "red", "stock": 12}`, écris une requête qui retourne les produits dont `stock` (en texte) est égal à `'12'`.

## Exercice 5 — CHECK constraint

Ajoute une contrainte `CHECK` sur une colonne `age INTEGER` d'une table `users` pour interdire toute valeur négative.
