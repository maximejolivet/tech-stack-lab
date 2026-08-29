# PostgreSQL

## 1. Introduction

PostgreSQL est un SGBDR open source réputé pour sa rigueur SQL et la richesse de son système de types. Ce dossier suppose [`../mysql/`](../mysql/) déjà vu et se concentre **sur ce qui diffère** de MySQL plutôt que de répéter les bases communes du SQL (CRUD, jointures, agrégation — déjà couvertes côté MySQL et transférables directement).

**À quoi sert-il ?**
- Les mêmes cas d'usage qu'un SGBDR classique (persistance transactionnelle, intégrité relationnelle), avec des besoins de modélisation plus riches : données semi-structurées (JSON), requêtes analytiques complexes, extensions spécialisées (géospatial, recherche plein texte).
- Souvent le choix par défaut dans l'écosystème Node.js/Python moderne (Prisma, Django ORM) et dans les offres cloud managées (Supabase, Neon, RDS).

**Où se situe-t-il dans une architecture web ?** Identique à MySQL — couche de persistance, accédée via un ORM ([`../django/`](../django/), [`../symfony/`](../symfony/) via Doctrine) ou des requêtes préparées.

**Avantages** : conformité SQL standard plus stricte, type `JSONB` performant et indexable pour du contenu semi-structuré, fonctions analytiques avancées natives (window functions, CTE récursives), extensible (PostGIS pour le géospatial, `pg_trgm` pour la recherche floue).
**Limites** : réplication et clustering historiquement plus complexes à opérer que MySQL (bien que largement comblé par les offres managées modernes), écosystème d'hébergement mutualisé low-cost historiquement moins répandu que MySQL.

## 2. Prérequis

- SQL de base et notions de SGBDR déjà acquises via [`../mysql/`](../mysql/) — ce dossier ne réexplique pas `SELECT`/`JOIN`/`GROUP BY`.
- Un client PostgreSQL installé (`psql`, ou un outil graphique comme TablePlus/DBeaver).

## 3. Rappel des bases 🟢

### 01 - Différences syntaxiques immédiates avec MySQL

**Explication** — La syntaxe SQL de base est quasi identique, mais quelques différences apparaissent dès les premières requêtes.

```sql
-- MySQL : AUTO_INCREMENT ; PostgreSQL : SERIAL (ou IDENTITY, recommandé aujourd'hui)
CREATE TABLE users (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE
);
```

**Bonne pratique** : préférer `GENERATED ALWAYS AS IDENTITY` (norme SQL) à `SERIAL` (spécificité historique PostgreSQL) sur un nouveau projet — comportement plus prévisible vis-à-vis des permissions et des séquences.

### 02 - Guillemets et sensibilité à la casse

**Explication** — Contrairement à MySQL, PostgreSQL respecte strictement la casse des identifiants **entre guillemets doubles**, et convertit automatiquement en minuscules tout identifiant non guillemeté.

```sql
CREATE TABLE "Users" (id INT); -- table nommée exactement "Users"
SELECT * FROM Users;           -- erreur : cherche "users" (minuscule), n'existe pas
SELECT * FROM "Users";         -- fonctionne
```

**Bonne pratique** : ne jamais utiliser de majuscules dans les noms de tables/colonnes — s'épargne ce piège entièrement en restant en `snake_case` minuscule partout, cohérent avec les conventions PostgreSQL par défaut.

### 03 - Types de données spécifiques

**Explication** — PostgreSQL propose des types que MySQL n'a pas nativement : `UUID`, `ARRAY`, `JSONB`, types réseau (`INET`), types de plage (`DATERANGE`).

```sql
CREATE TABLE articles (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    tags TEXT[],
    metadata JSONB
);

INSERT INTO articles (tags, metadata) VALUES
    (ARRAY['sql', 'postgres'], '{"views": 120, "featured": true}');
```

### 04 - RETURNING

**Explication** — `INSERT`/`UPDATE`/`DELETE` peuvent retourner directement les lignes affectées, sans `SELECT` séparé.

```sql
INSERT INTO users (email) VALUES ('alice@test.com') RETURNING id, created_at;
```

**Bonne pratique** : utiliser `RETURNING` pour récupérer un ID généré ou une valeur calculée par défaut (timestamp) en un seul aller-retour réseau, plutôt qu'un `INSERT` suivi d'un `SELECT` séparé (`LAST_INSERT_ID()` côté MySQL).

### 05 - Requêtes JSONB de base

**Explication** — `JSONB` (binaire, indexable) est presque toujours préférable à `JSON` (texte brut, réévalué à chaque lecture).

```sql
SELECT metadata->>'views' AS views FROM articles WHERE metadata->>'featured' = 'true';
-- ->  retourne du JSONB, ->> retourne du texte
```

**Erreur fréquente** : utiliser le type `JSON` par défaut au lieu de `JSONB` — `JSON` stocke le texte brut tel quel (plus rapide à écrire, mais reparsé à chaque requête et non indexable efficacement), alors que `JSONB` est quasiment toujours le bon choix pour des données interrogées.

### 06 - Contraintes CHECK

**Explication** — Contrainte de validation au niveau de la colonne/table, plus riche que MySQL historiquement (où `CHECK` était longtemps ignoré silencieusement — corrigé depuis MySQL 8.0.16, mais moins central dans la culture MySQL).

```sql
CREATE TABLE products (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    price NUMERIC(10,2) NOT NULL CHECK (price >= 0)
);
```

## 4. Concepts intermédiaires 🟡

- **Window functions** : calculer une valeur agrégée "par-dessus" chaque ligne sans réduire le nombre de lignes du résultat (contrairement à `GROUP BY`).

```sql
SELECT
    customer_id,
    order_date,
    total,
    SUM(total) OVER (PARTITION BY customer_id ORDER BY order_date) AS cumul_client
FROM orders;
```

- **CTE (Common Table Expressions)** : nommer une sous-requête intermédiaire lisible via `WITH`, y compris de façon **récursive** (ex. parcourir une hiérarchie de catégories parent/enfant).

```sql
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, c.parent_id
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree;
```

- **Index sur JSONB (GIN)** : un index B-Tree classique ne permet pas d'interroger efficacement l'intérieur d'un `JSONB` — un index `GIN` le rend possible.

```sql
CREATE INDEX idx_articles_metadata ON articles USING GIN (metadata);
```

- **Index partiels** : indexer seulement un sous-ensemble de lignes correspondant à une condition fréquente, plus petit et plus rapide qu'un index complet.

```sql
CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status = 'pending';
```

- **UPSERT (`ON CONFLICT`)** : insérer ou mettre à jour en une seule requête atomique, équivalent PostgreSQL de `ON DUPLICATE KEY UPDATE` côté MySQL, avec une syntaxe plus expressive.

```sql
INSERT INTO users (email, last_seen) VALUES ('alice@test.com', now())
ON CONFLICT (email) DO UPDATE SET last_seen = EXCLUDED.last_seen;
```

- **Vues matérialisées** : équivalent d'une vue classique mais dont le résultat est **calculé et stocké physiquement**, à rafraîchir explicitement (`REFRESH MATERIALIZED VIEW`) — utile pour des agrégations coûteuses consultées fréquemment mais pas nécessairement à jour à la seconde près.

## 5. Concepts avancés 🟠🔴

- **MVCC et différences de verrouillage avec MySQL** : PostgreSQL implémente le MVCC en conservant plusieurs versions physiques d'une même ligne (chaque `UPDATE` crée une nouvelle version, l'ancienne devient obsolète), nettoyées ensuite par le processus `VACUUM`. C'est différent de l'implémentation InnoDB de MySQL (undo logs). Conséquence pratique : des tables PostgreSQL fortement mises à jour peuvent grossir en espace disque ("bloat") tant que `VACUUM` (automatique via `autovacuum`, ou manuel) n'a pas nettoyé les anciennes versions.

**Erreur fréquente** : désactiver `autovacuum` pour "gagner en performance d'écriture" — sur une table à forte volumétrie d'updates, cela accumule du bloat qui dégrade progressivement les performances de lecture ET d'écriture, jusqu'à nécessiter une opération de maintenance lourde (`VACUUM FULL`, qui verrouille la table).

- **Niveaux d'isolation** : PostgreSQL implémente réellement les 4 niveaux SQL standard (`READ COMMITTED` par défaut, `REPEATABLE READ`, `SERIALIZABLE` — véritablement sérialisable, contrairement à certaines implémentations partielles ailleurs), avec détection automatique des conflits de sérialisation qui font échouer une transaction (à retenter côté applicatif) plutôt que de produire un résultat incohérent silencieusement.
- **Extensions** : PostgreSQL s'étend nativement via des extensions officielles ou tierces — `PostGIS` (données géospatiales), `pg_trgm` (recherche floue/similarité de texte), `uuid-ossp`/`pgcrypto` (génération d'UUID), `TimescaleDB` (séries temporelles). C'est un avantage architectural majeur sur MySQL : une seule base peut couvrir des besoins qui nécessiteraient ailleurs un système tiers dédié.
- **Requêtes analytiques et window functions avancées** : `RANK()`, `ROW_NUMBER()`, `LAG()`/`LEAD()` pour comparer une ligne à la précédente/suivante dans un ordre donné — cas d'usage type : détecter des ruptures de séquence, calculer un classement, comparer une valeur à celle du mois précédent.
- **Réplication logique** : au-delà de la réplication physique (streaming, proche de MySQL), PostgreSQL supporte la réplication **logique** (`pg_logical`, `CREATE PUBLICATION`/`SUBSCRIPTION`) — réplique des changements au niveau ligne plutôt que fichier binaire, permettant de répliquer sélectivement certaines tables ou vers des versions PostgreSQL différentes.
- **`EXPLAIN ANALYZE`** : comme `EXPLAIN` côté MySQL, mais exécute réellement la requête et compare le plan estimé au comportement réel (temps effectif par nœud du plan) — diagnostic plus précis qu'un plan purement théorique.

## 6. Commandes / syntaxe à connaître

```sql
\l              -- lister les bases (psql)
\dt             -- lister les tables
\d nom_table    -- décrire une table
\di             -- lister les index
EXPLAIN ANALYZE SELECT ...;
```

```bash
pg_dump mon_site > backup.sql     # export
psql mon_site < backup.sql         # import
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Migration d'un schéma MySQL vers PostgreSQL avec enrichissement**

En partant du schéma boutique du mini-projet MySQL (`products`, `customers`, `orders`, `order_items`) :
- Migrer les types (`AUTO_INCREMENT` → `IDENTITY`), et ajouter une colonne `metadata JSONB` sur `products` pour des attributs variables selon la catégorie (ex. taille pour un vêtement, capacité pour un appareil électronique).
- Écrire une requête utilisant une window function qui calcule, pour chaque commande, le cumul des dépenses du client jusqu'à cette commande incluse.
- Ajouter un index `GIN` sur `metadata` et écrire une requête qui filtre les produits par un attribut JSONB donné.
- Implémenter un `ON CONFLICT` pour un import idempotent de produits (mise à jour du prix si le produit existe déjà, identifié par une référence unique).
- Bonus : écrire une CTE récursive pour une arborescence de catégories de produits, et comparer le plan `EXPLAIN ANALYZE` de la requête de listing des commandes entre la version normalisée et une version dénormalisée.

## Checklist

- [ ] Comprendre les différences syntaxiques et de types avec MySQL
- [ ] Savoir utiliser `JSONB`, `RETURNING`, `ON CONFLICT`
- [ ] Maîtriser les window functions et les CTE (y compris récursives)
- [ ] Comprendre les concepts importants (index GIN/partiels, vues matérialisées)
- [ ] Savoir debugger (`EXPLAIN ANALYZE`, `\d`, logs)
- [ ] Connaître les bonnes pratiques (éviter les identifiants avec majuscules, `autovacuum` actif, `JSONB` plutôt que `JSON`)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (MVCC/bloat, isolation SERIALIZABLE, extensions, réplication logique)

## 10. Ressources

- [Documentation officielle PostgreSQL](https://www.postgresql.org/docs/) — référence complète, très détaillée.
- [PostgreSQL Exercises](https://pgexercises.com/) — exercices interactifs en ligne, bon complément pratique.
- [roadmap.sh — SQL](https://roadmap.sh/sql) et [roadmap.sh — PostgreSQL DBA](https://roadmap.sh/postgresql-dba) — vues d'ensemble des compétences.
- [`../mysql/`](../mysql/) pour la comparaison directe avec le premier SGBDR couvert dans ce repo.
