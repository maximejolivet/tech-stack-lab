# MySQL

## 1. Introduction

MySQL est un système de gestion de base de données relationnelle (SGBDR) open source, le plus utilisé historiquement dans l'écosystème PHP/web (LAMP). Ce dossier suppose une familiarité de base avec le SQL et se concentre sur MySQL spécifiquement : son moteur de stockage, ses spécificités de verrouillage, et les bonnes pratiques de conception/performance. Les frameworks vus dans ce repo ([`../laravel/`](../laravel/) via Eloquent, [`../symfony/`](../symfony/) via Doctrine, [`../django/`](../django/) via son ORM) s'appuient dessus sans en masquer totalement le fonctionnement.

**À quoi sert-il ?**
- Stocker des données structurées de façon durable, avec des garanties de cohérence (transactions, contraintes).
- Interroger efficacement de gros volumes de données via un langage déclaratif (SQL) plutôt que du code impératif.
- Modéliser des relations entre entités (utilisateurs, commandes, produits) via des clés étrangères et des jointures.

**Où se situe-t-il dans une architecture web ?** Couche de persistance, généralement derrière une couche applicative (PHP, Node.js) qui ne s'y connecte jamais directement depuis le navigateur — accédé via un ORM ([`../laravel/`](../laravel/), [`../symfony/`](../symfony/)) ou des requêtes préparées manuelles.

**Avantages** : garanties ACID sur les transactions, écosystème d'outillage immense, performances solides en lecture pour la majorité des cas d'usage web, hébergement disponible partout.
**Limites** : moins riche que PostgreSQL sur certains types de données avancés (JSON, tableaux) et certaines fonctionnalités SQL (voir [`../postgresql/`](../postgresql/) pour la comparaison), le moteur de stockage par défaut (InnoDB) verrouille différemment de PostgreSQL — des pièges de concurrence différents à connaître.

## 2. Prérequis

- Notions de base de PHP ou d'un autre langage backend pour exécuter des requêtes depuis du code (voir [`../php/`](../php/)).
- Un client MySQL installé (`mysql` CLI, ou un outil graphique comme TablePlus/DBeaver).
- Aucune connaissance SQL préalable n'est strictement nécessaire — ce dossier part des bases.

## 3. Rappel des bases 🟢

### 01 - Connexion et création d'une base

**Explication** — Se connecter au serveur, créer une base et une table.

```sql
mysql -u root -p

CREATE DATABASE mon_site CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE mon_site;
```

**Bonne pratique** : toujours utiliser `utf8mb4` (et non `utf8`, qui est en réalité limité à 3 octets par caractère dans MySQL) — `utf8mb4` seul supporte correctement les emojis et l'ensemble d'Unicode.

### 02 - Création de table et types

**Explication** — `CREATE TABLE` définit les colonnes, leurs types, et les contraintes.

```sql
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Bonne pratique** : choisir le type le plus étroit qui couvre le besoin réel (`INT UNSIGNED` plutôt que `BIGINT` si le volume ne le justifie pas) — impacte directement la taille des index et donc la performance.

### 03 - CRUD de base

**Explication** — Les quatre opérations fondamentales.

```sql
INSERT INTO users (email) VALUES ('alice@test.com');
SELECT id, email FROM users WHERE email = 'alice@test.com';
UPDATE users SET email = 'alice2@test.com' WHERE id = 1;
DELETE FROM users WHERE id = 1;
```

**Erreur fréquente** : exécuter un `UPDATE`/`DELETE` sans clause `WHERE` — modifie ou supprime **toutes** les lignes de la table. Toujours tester le `WHERE` avec un `SELECT` équivalent avant d'exécuter la modification.

### 04 - Clés primaires et étrangères

**Explication** — La clé étrangère garantit l'intégrité référentielle : impossible d'insérer une ligne pointant vers une entité inexistante.

```sql
CREATE TABLE orders (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Bonne pratique** : choisir `ON DELETE CASCADE`/`RESTRICT`/`SET NULL` de façon délibérée selon la sémantique métier (supprimer les commandes d'un utilisateur supprimé, ou interdire la suppression tant qu'il a des commandes) — ne jamais laisser le comportement par défaut sans y avoir réfléchi.

### 05 - Jointures

**Explication** — Combiner des lignes de plusieurs tables selon une condition de correspondance.

```sql
SELECT orders.id, users.email
FROM orders
INNER JOIN users ON users.id = orders.user_id;

-- LEFT JOIN conserve les users même sans commande (order.* sera NULL)
SELECT users.email, orders.id
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

**Erreur fréquente** : utiliser `INNER JOIN` quand on veut aussi les lignes sans correspondance (ex. "tous les utilisateurs, avec leurs commandes s'ils en ont") — `INNER JOIN` élimine silencieusement les utilisateurs sans commande, `LEFT JOIN` est ici le bon choix.

### 06 - Agrégation et GROUP BY

**Explication** — Calculer des statistiques sur des groupes de lignes.

```sql
SELECT user_id, COUNT(*) AS nb_commandes, SUM(total) AS montant_total
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 3;
```

**Erreur fréquente** : filtrer sur une colonne agrégée avec `WHERE` au lieu de `HAVING` — `WHERE` s'applique **avant** l'agrégation (sur les lignes brutes), `HAVING` s'applique **après** (sur les groupes), donc `WHERE COUNT(*) > 3` est une erreur de syntaxe.

## 4. Concepts intermédiaires 🟡

- **Index** : structure de données (B-Tree par défaut) qui accélère la recherche sur une colonne, au prix d'un coût d'écriture supplémentaire (chaque `INSERT`/`UPDATE` doit aussi mettre à jour l'index).

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE UNIQUE INDEX idx_users_email ON users(email); -- combine index + contrainte d'unicité
```

**Erreur fréquente** : indexer toutes les colonnes "au cas où" — chaque index ralentit les écritures et consomme de l'espace disque/mémoire ; n'indexer que les colonnes réellement utilisées dans des `WHERE`, `JOIN` ou `ORDER BY` fréquents.

- **EXPLAIN** : analyse le plan d'exécution réel d'une requête (scan complet de table vs utilisation d'un index) — outil de diagnostic indispensable avant d'optimiser à l'aveugle.

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 42;
-- "type: ALL" = scan complet de table (mauvais signe sur une grosse table)
-- "type: ref"/"const" = utilisation efficace d'un index
```

- **Transactions** : regrouper plusieurs opérations en une unité atomique — soit toutes appliquées, soit aucune.

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- ou ROLLBACK en cas d'erreur applicative
```

**Bonne pratique** : toute opération qui modifie plusieurs tables de façon interdépendante (ex. transfert d'argent, création de commande + décrément de stock) doit être englobée dans une transaction, sinon une erreur en cours de route laisse les données dans un état incohérent.

- **Normalisation** : organiser le schéma pour éviter la duplication de données (formes normales 1NF/2NF/3NF) — réduit les risques d'incohérence, au prix de davantage de jointures. La **dénormalisation** volontaire (dupliquer une donnée pour éviter une jointure coûteuse) reste un compromis de performance légitime sur des cas identifiés, pas un défaut de conception par défaut.
- **Requêtes préparées** : paramétrer une requête plutôt que concaténer des valeurs dans la chaîne SQL — protection native contre l'injection SQL, à utiliser systématiquement dès qu'une valeur vient d'une entrée utilisateur (voir [`../security/`](../security/), [`../php/`](../php/) pour PDO).

## 5. Concepts avancés 🟠🔴

- **Moteurs de stockage** : InnoDB (par défaut depuis MySQL 5.5) supporte les transactions et les clés étrangères ; MyISAM (historique) ne les supporte pas mais reste parfois utilisé pour des cas en lecture seule très spécifiques. En pratique, InnoDB est le choix par défaut correct dans la quasi-totalité des projets modernes.
- **Verrouillage et niveaux d'isolation** : InnoDB utilise du verrouillage au niveau ligne (row-level locking) et le MVCC (Multi-Version Concurrency Control) pour permettre des lectures concurrentes sans bloquer les écritures. Les niveaux d'isolation (`READ COMMITTED`, `REPEATABLE READ` — défaut MySQL, `SERIALIZABLE`) déterminent quels phénomènes de concurrence (lecture sale, lecture non répétable, phantom read) sont possibles.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**Erreur fréquente** : ignorer les deadlocks en production — deux transactions qui verrouillent des lignes dans un ordre inversé peuvent se bloquer mutuellement ; MySQL détecte le deadlock et annule automatiquement l'une des deux transactions (à retenter côté applicatif), mais un ordre de verrouillage cohérent (toujours verrouiller les tables/lignes dans le même ordre) réduit fortement leur fréquence.

- **Réplication** : un serveur primaire (writes) réplique ses changements vers un ou plusieurs serveurs secondaires (reads) — répartit la charge de lecture et fournit une base pour la haute disponibilité. La réplication est asynchrone par défaut (léger délai de propagation), à connaître pour éviter de lire une donnée "en retard" juste après une écriture sur le primaire.
- **Partitionnement** : diviser une très grosse table en partitions physiques (par plage de dates, par hash) pour accélérer certaines requêtes et faciliter la purge de données anciennes — pertinent seulement à partir d'une volumétrie significative, pas un réflexe par défaut.
- **Sauvegardes** : `mysqldump` pour des bases de taille modeste, outils physiques (Percona XtraBackup) pour des bases volumineuses où un dump logique serait trop lent à restaurer ; toujours tester une restauration réelle, pas seulement vérifier que la sauvegarde "s'exécute sans erreur".
- **Query cache et buffer pool** : le buffer pool InnoDB (`innodb_buffer_pool_size`) met en cache les pages de données/index en mémoire — son dimensionnement est le levier de performance le plus impactant sur un serveur dédié à MySQL, généralement réglé à 60-70% de la RAM disponible.

## 6. Commandes / syntaxe à connaître

```sql
SHOW DATABASES;
SHOW TABLES;
DESCRIBE users;                        -- structure d'une table
SHOW INDEX FROM users;
EXPLAIN SELECT ...;                    -- plan d'exécution
SHOW PROCESSLIST;                      -- requêtes en cours
```

```bash
mysqldump -u root -p mon_site > backup.sql   # export
mysql -u root -p mon_site < backup.sql        # import
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Schéma et requêtes pour une boutique en ligne simplifiée**

- Concevoir un schéma normalisé avec `products`, `customers`, `orders`, `order_items` (clés étrangères correctes, contraintes `NOT NULL`/`UNIQUE` pertinentes).
- Écrire une requête qui liste, par client, le montant total dépensé, triée du plus gros au plus petit acheteur (`JOIN` + `GROUP BY`).
- Écrire une transaction qui crée une commande et décrémente le stock du produit, avec `ROLLBACK` si le stock est insuffisant.
- Ajouter les index nécessaires pour que la requête de listing des commandes d'un client reste rapide sur une table à 1 million de lignes, et le vérifier avec `EXPLAIN`.
- Bonus : mettre en place une réplique en lecture seule (via Docker) et discuter dans quel cas une lecture juste après écriture poserait problème.

## Checklist

- [ ] Comprendre les fondamentaux (CRUD, types, clés primaires/étrangères, jointures)
- [ ] Savoir créer un schéma normalisé et écrire des requêtes de base
- [ ] Maîtriser la syntaxe principale (agrégation, sous-requêtes, transactions)
- [ ] Comprendre les concepts importants (index, `EXPLAIN`, requêtes préparées)
- [ ] Savoir debugger (`EXPLAIN`, `SHOW PROCESSLIST`, logs lents)
- [ ] Connaître les bonnes pratiques (`utf8mb4`, transactions sur écritures liées, éviter les scans complets)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (verrouillage/MVCC, réplication, partitionnement)

## 10. Ressources

- [Documentation officielle MySQL](https://dev.mysql.com/doc/) — référence complète.
- [Use The Index, Luke](https://use-the-index-luke.com/) — ressource de référence sur les index et la performance SQL, applicable au-delà de MySQL.
- [roadmap.sh — SQL](https://roadmap.sh/sql) — vue d'ensemble des compétences SQL couvrant MySQL/PostgreSQL.
- [`../postgresql/`](../postgresql/) pour la comparaison directe avec un second SGBDR.
