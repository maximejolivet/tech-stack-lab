# Solutions — Niveau 1 (Bases)

## Exercice 1

```sql
CREATE TABLE articles (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title TEXT NOT NULL,
    tags TEXT[]
);
```

## Exercice 2

```sql
INSERT INTO articles (title) VALUES ('Mon premier article') RETURNING id;
```

## Exercice 3

`"Articles"` a été créée avec des guillemets doubles, donc PostgreSQL respecte strictement la casse exacte `Articles`. `SELECT * FROM Articles` (sans guillemets) est automatiquement converti en minuscules par PostgreSQL, donc cherche une table `articles` qui n'existe pas.

```sql
SELECT * FROM "Articles"; -- avec guillemets, casse respectée
```

## Exercice 4

```sql
SELECT * FROM products WHERE metadata->>'stock' = '12';
```

## Exercice 5

```sql
ALTER TABLE users ADD CONSTRAINT check_age_positive CHECK (age >= 0);
```
