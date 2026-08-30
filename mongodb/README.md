# MongoDB

## 1. Introduction

MongoDB est une base de données NoSQL **orientée documents** : les données sont stockées sous forme de documents BSON (variante binaire de JSON) regroupés en **collections**, plutôt qu'en lignes structurées dans des tables comme [`../mysql/`](../mysql/) ou [`../postgresql/`](../postgresql/). Ce dossier suppose ces deux SGBDR déjà vus et se concentre sur ce qui **diffère structurellement** d'un modèle relationnel, plutôt que de réexpliquer les notions de base de données déjà couvertes (CRUD, index, transactions).

**À quoi sert-il ?**
- Modéliser des données dont la structure varie d'un document à l'autre ou évolue fréquemment (contenu éditorial, catalogues produits hétérogènes, profils utilisateurs avec des champs optionnels nombreux) sans migration de schéma à chaque changement.
- Absorber un fort volume d'écritures avec une structure de document proche de celle manipulée côté application (JSON), réduisant l'impédance entre le code et la base.
- Prototyper rapidement quand le schéma final n'est pas encore figé.

**Où se situe-t-il dans une architecture web ?** Couche de persistance, au même niveau que MySQL/PostgreSQL, accédée via un driver officiel ou un ODM (*Object-Document Mapper*) — Mongoose est le standard de facto côté [`../nodejs/`](../nodejs/), jouant un rôle proche de celui d'un ORM (Doctrine, Eloquent) mais adapté à un schéma flexible plutôt que rigide.

**Avantages**
- Schéma flexible : deux documents d'une même collection peuvent avoir des champs différents, utile quand le modèle de données évolue vite ou varie légitimement selon le sous-type d'entité.
- Modèle de document proche de l'objet manipulé côté application — souvent moins de traduction/mapping qu'avec un modèle relationnel normalisé.
- Scalabilité horizontale native (sharding) pensée dès la conception, sans devoir la rajouter après coup comme c'est souvent le cas en SQL.

**Limites**
- Pas de jointures natives entre collections (contrairement au `JOIN` SQL déjà vu) : les relations doivent être modélisées par **embedding** ou **référencement** applicatif, ce qui déplace une partie de la charge d'intégrité vers le code.
- Les garanties transactionnelles multi-documents existent (depuis MongoDB 4.0) mais restent plus coûteuses en performance et moins centrales dans l'usage courant qu'une transaction SQL — voir section 4.
- Un schéma "flexible" n'est pas un schéma "absent" : sans discipline (validation, conventions d'équipe), une collection peut dériver vers une incohérence difficile à corriger a posteriori.

## 2. Prérequis

- Notions de base de SGBDR déjà acquises via [`../mysql/`](../mysql/) — ce dossier ne réexplique pas ce qu'est une clé primaire, un index ou une transaction, seulement ce qui change dans un modèle document.
- Un langage backend pour interroger MongoDB en dehors du shell — Node.js ([`../nodejs/`](../nodejs/)) est l'appariement le plus naturel et le plus documenté.
- MongoDB installé localement, ou un cluster gratuit sur MongoDB Atlas ; le client `mongosh` (shell interactif) ou l'outil graphique MongoDB Compass.

## 3. Rappel des bases 🟢

### 01 - Vocabulaire : documents, collections, bases

**Explication** — Une **base de données** contient des **collections**, elles-mêmes composées de **documents** — l'équivalent conceptuel d'une base SQL contenant des tables composées de lignes, mais un document est une structure arborescente (BSON), pas une ligne à colonnes fixes.

```javascript
// Vocabulaire équivalent
// SQL          → MongoDB
// database     → database
// table        → collection
// row          → document
// column       → field
```

**Cas d'usage** : garder ce mapping en tête pour transférer les réflexes déjà acquis en SQL, tout en restant vigilant sur les endroits où l'analogie s'arrête (pas de schéma fixe, pas de jointure native).

### 02 - Connexion et création d'une base/collection

**Explication** — MongoDB crée une base et une collection **implicitement**, à la première écriture — pas de `CREATE DATABASE`/`CREATE TABLE` obligatoire au préalable.

```javascript
mongosh

use mon_site               // bascule sur (et crée si besoin) la base "mon_site"
db.products.insertOne({ name: "Clavier", price: 45 })  // crée la collection "products" au passage
```

**Erreur fréquente** : s'attendre à voir la base apparaître dans `show dbs` immédiatement après un `use` sans écriture — MongoDB ne matérialise une base vide qu'à la première insertion réelle de document.

### 03 - Insertion de documents

**Explication** — `insertOne`/`insertMany` acceptent directement des objets JSON, sans déclaration de structure préalable.

```javascript
db.products.insertOne({ name: "Clavier", price: 45, tags: ["informatique", "périphérique"] })

db.products.insertMany([
    { name: "Souris", price: 15 },
    { name: "Écran", price: 199, tags: ["informatique"] }
])
```

**Bonne pratique** : même si rien ne l'impose techniquement, garder une structure de champs cohérente entre documents d'une même collection par convention d'équipe — c'est ce qui évite qu'un schéma "flexible" ne devienne un schéma "chaotique" (voir la validation en section 4).

### 04 - Requêtes de base (find et opérateurs de comparaison)

**Explication** — `find()` accepte un objet de filtre ; les opérateurs de comparaison (`$gt`, `$lt`, `$in`...) remplacent les opérateurs SQL (`>`, `<`, `IN`).

```javascript
db.products.find({ price: { $gt: 20 } })                 // équivalent de WHERE price > 20
db.products.find({ price: { $gte: 20, $lte: 200 } })       // équivalent de WHERE price BETWEEN 20 AND 200
db.products.find({ category: { $in: ["informatique", "bureau"] } })  // équivalent de WHERE category IN (...)
db.products.find({ name: "Clavier" }).sort({ price: -1 })    // ORDER BY price DESC
```

**Cas d'usage** : `findOne()` retourne directement un seul document (ou `null`) plutôt qu'un curseur, pratique quand on sait qu'au plus un résultat est attendu (recherche par identifiant unique).

### 05 - Mise à jour de documents

**Explication** — `updateOne`/`updateMany` nécessitent un opérateur de mise à jour (`$set`, `$inc`...) — écrire directement `{ price: 50 }` sans `$set` **remplacerait tout le document** par ce seul champ, un piège fréquent.

```javascript
db.products.updateOne({ name: "Clavier" }, { $set: { price: 50 } })
db.products.updateMany({ category: "électronique" }, { $inc: { price: 5 } })  // +5 sur chaque document matché
```

**Erreur fréquente** : appeler `updateOne({ _id: id }, { price: 50 })` sans `$set` — MongoDB interprète le second argument comme le **document de remplacement complet**, supprimant tous les autres champs du document ciblé.

### 06 - Suppression de documents

**Explication** — `deleteOne`/`deleteMany` suivent la même logique de filtre que `find`.

```javascript
db.products.deleteOne({ name: "Clavier" })
db.products.deleteMany({ price: { $lt: 10 } })
```

**Erreur fréquente** : appeler `deleteMany({})` en pensant filtrer sur un objet vide "par défaut" — un filtre vide matche **tous** les documents de la collection, supprimant l'intégralité de son contenu (équivalent d'un `DELETE FROM table` sans `WHERE`).

### 07 - Le champ `_id` et `ObjectId`

**Explication** — Chaque document possède un champ `_id`, unique au sein de sa collection, généré automatiquement comme un `ObjectId` (12 octets encodant notamment un timestamp) si non fourni explicitement — l'équivalent fonctionnel d'une clé primaire auto-incrémentée, mais généré côté client/driver plutôt que par la base.

```javascript
db.products.insertOne({ name: "Clavier" })
// _id: ObjectId("665f1a2b3c4d5e6f7a8b9c0d") généré automatiquement

db.products.findOne({ _id: ObjectId("665f1a2b3c4d5e6f7a8b9c0d") })
```

**Bonne pratique** : ne jamais trier ou filtrer sur `_id` en supposant un ordre séquentiel strict comme avec un `AUTO_INCREMENT` SQL — l'`ObjectId` encode un timestamp approximatif à la seconde, pas un compteur exact.

### 08 - Embedding vs référencement

**Explication** — Sans jointure native, modéliser une relation impose un choix explicite : **embedder** (imbriquer) les données liées directement dans le document parent, ou les **référencer** par un identifiant vers une autre collection (relation applicative, résolue par une requête séparée ou `$lookup`, voir section 4).

```javascript
// Embedding : l'adresse fait partie intégrante du document utilisateur, jamais consultée seule
db.users.insertOne({
    name: "Max",
    address: { street: "1 rue de Paris", city: "Paris" }
})

// Référencement : les commandes sont nombreuses, interrogées indépendamment, et grossiraient le document utilisateur indéfiniment
db.orders.insertOne({ userId: ObjectId("..."), total: 99.90 })
```

**Cas d'usage** : embedder une relation "possédée" et bornée (adresse, préférences), référencer une relation qui grandit sans limite ou qui doit être interrogée/mise à jour indépendamment de son parent (commandes d'un client, commentaires d'un article très commenté).

### 09 - Index de base

**Explication** — Comme en SQL, un index accélère la recherche sur un champ fréquemment filtré, au prix d'un coût d'écriture supplémentaire.

```javascript
db.products.createIndex({ name: 1 })              // index ascendant sur "name"
db.users.createIndex({ email: 1 }, { unique: true })  // index + contrainte d'unicité
```

## 4. Concepts intermédiaires 🟡

- **Pipeline d'agrégation** : équivalent fonctionnel du `GROUP BY`/`JOIN` SQL vu dans [`../mysql/`](../mysql/), construit comme une succession d'étapes (`$match` filtre, `$group` agrège, `$sort` trie, `$project` reforme la sortie) appliquées dans l'ordre.

```javascript
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
    { $sort: { total: -1 } }
])
// équivalent de : SELECT customer_id, SUM(amount) AS total FROM orders
//                 WHERE status = 'completed' GROUP BY customer_id ORDER BY total DESC
```

- **`$lookup` (jointure d'agrégation)** : rapproche deux collections référencées au sein d'un pipeline, l'équivalent le plus proche d'un `JOIN` — à utiliser avec discernement, ce n'est pas optimisé de la même façon qu'une jointure SQL native sur un modèle normalisé.

```javascript
db.orders.aggregate([
    { $lookup: { from: "customers", localField: "customerId", foreignField: "_id", as: "customer" } }
])
```

- **Validation de schéma (JSON Schema)** : MongoDB est *schema-flexible*, pas *schema-less* en pratique — un validateur JSON Schema attaché à une collection impose des règles minimales (champs requis, types) sans revenir à la rigidité complète d'un schéma SQL.

```javascript
db.createCollection("products", {
    validator: {
        $jsonSchema: {
            required: ["name", "price"],
            properties: {
                name: { bsonType: "string" },
                price: { bsonType: "number", minimum: 0 }
            }
        }
    }
})
```

- **Opérateurs de requête sur tableaux** : `$elemMatch` (au moins un élément du tableau satisfait plusieurs conditions simultanément), `$all` (le tableau contient tous les éléments listés) — nécessaires dès qu'un champ est un tableau de sous-documents, un cas très courant en modèle document.

```javascript
db.products.find({ reviews: { $elemMatch: { rating: { $gte: 4 }, verified: true } } })
db.products.find({ tags: { $all: ["informatique", "promo"] } })
```

- **Transactions multi-documents** : depuis MongoDB 4.0, une transaction ACID peut englober plusieurs documents (et depuis 4.2, plusieurs collections/bases), avec la même sémantique `commit`/`abort` qu'une transaction SQL déjà vue dans [`../mysql/`](../mysql/) — historiquement absentes de MongoDB, elles restent plus coûteuses en performance qu'une simple écriture atomique sur un document unique (garantie de base, sans transaction, dès qu'une opération ne touche qu'un seul document).

```javascript
const session = db.getMongo().startSession()
session.startTransaction()
try {
    db.accounts.updateOne({ _id: 1 }, { $inc: { balance: -50 } }, { session })
    db.accounts.updateOne({ _id: 2 }, { $inc: { balance: 50 } }, { session })
    session.commitTransaction()
} catch (e) {
    session.abortTransaction()
}
```

- **ODM avec Mongoose** (Node.js) : définit un schéma applicatif (types, champs requis, valeurs par défaut) au-dessus d'une collection nativement flexible, ajoute validation et middlewares (`pre`/`post` save) — rôle comparable à un ORM, mais le schéma vit côté application, pas côté base.

```javascript
const productSchema = new mongoose.Schema({
    name: { type: String, required: true },
    price: { type: Number, required: true, min: 0 }
})
const Product = mongoose.model("Product", productSchema)
```

## 5. Concepts avancés 🟠🔴

- **Sharding** : répartition horizontale des données d'une collection sur plusieurs serveurs (*shards*) selon une **clé de sharding** choisie à l'avance — approche différente du partitionnement MySQL/PostgreSQL déjà vu (généralement mono-serveur), ici pensée nativement pour distribuer la charge sur plusieurs machines. Le choix de la clé de sharding est structurant et difficile à changer après coup : une mauvaise clé (peu de valeurs distinctes, écritures concentrées sur une valeur récente comme un timestamp) crée des "hot shards" qui annulent le bénéfice de la distribution.
- **Replica sets** : un ensemble de serveurs (un primaire, plusieurs secondaires) répliquant les mêmes données, avec **basculement automatique** (élection d'un nouveau primaire) en cas de panne — le mécanisme de haute disponibilité est intégré nativement au protocole, alors qu'un failover MySQL/PostgreSQL nécessite généralement un outil tiers (Orchestrator, Patroni) au-dessus de la réplication de base.
- **Moteur de stockage WiredTiger** : moteur par défaut depuis MongoDB 3.2, avec verrouillage au niveau document (pas au niveau collection) et compression native — comparable en esprit à InnoDB côté MySQL (verrouillage fin, MVCC), mais organisé autour de documents plutôt que de lignes/pages relationnelles.
- **Index composés et stratégie d'agrégation** : un index composé (`{ status: 1, createdAt: -1 }`) doit suivre l'ordre des champs réellement filtrés/triés dans les requêtes et pipelines fréquents — comme en SQL, l'ordre des colonnes d'un index composé change radicalement son utilité selon la requête.
- **Quand ne PAS utiliser MongoDB** : des données fortement relationnelles nécessitant de nombreuses jointures cohérentes entre de multiples entités (comptabilité, systèmes bancaires, tout ce qui exige des contraintes d'intégrité référentielle strictes et vérifiées par la base elle-même) restent mieux servies par [`../mysql/`](../mysql/) ou [`../postgresql/`](../postgresql/) — MongoDB déplace cette responsabilité vers l'application, un compromis délibéré à ne pas ignorer par simple préférence de syntaxe JSON.

## 6. Commandes / syntaxe à connaître

```javascript
show dbs                          // lister les bases
use mon_site                       // basculer sur une base (la crée si besoin, à la 1ère écriture)
show collections                    // lister les collections de la base courante

db.products.find()                    // tous les documents
db.products.findOne({ _id: ... })       // un seul document
db.products.insertOne({ ... })            // insérer un document
db.products.updateOne({ ... }, { $set: { ... } })  // mettre à jour (avec opérateur !)
db.products.deleteOne({ ... })              // supprimer
db.products.createIndex({ champ: 1 })         // créer un index
db.products.explain().find({ ... })             // plan d'exécution d'une requête
```

```bash
mongodump --db=mon_site --out=./backup    # export
mongorestore --db=mon_site ./backup/mon_site  # import
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Modélisation document d'une boutique en ligne simplifiée**

Reprendre le mini-projet du dossier MySQL ([`../mysql/README.md`](../mysql/README.md) : `products`, `customers`, `orders`, `order_items`) et le remodéliser en MongoDB, en prenant des décisions explicites plutôt qu'en transposant littéralement le schéma relationnel :
- Décider ce qui doit être **embedded** (les `order_items` d'une commande, bornés et jamais consultés hors de leur commande) et ce qui doit être **référencé** (le client d'une commande, consulté indépendamment et partagé entre plusieurs commandes).
- Écrire un pipeline d'agrégation qui calcule, par client, le montant total dépensé — équivalent document du `JOIN` + `GROUP BY` du mini-projet MySQL.
- Ajouter un validateur JSON Schema sur `products` garantissant `name` et `price` requis, `price` positif.
- Écrire une transaction multi-documents qui crée une commande et décrémente le stock du produit correspondant, avec `abortTransaction()` explicite si le stock est insuffisant.
- Bonus : comparer en 3-4 lignes le nombre de requêtes nécessaires pour afficher une commande complète (client + produits) dans chaque modèle, et discuter dans quel cas l'un devient plus coûteux que l'autre à l'échelle.

## Checklist

- [ ] Comprendre les fondamentaux (documents, collections, `_id`/ObjectId, CRUD)
- [ ] Savoir modéliser une relation en choisissant entre embedding et référencement
- [ ] Maîtriser la syntaxe principale (opérateurs de requête, `$set`/`$inc`, index)
- [ ] Comprendre les concepts importants (pipeline d'agrégation, `$lookup`, validation de schéma)
- [ ] Savoir debugger (`.explain()`, vérifier l'utilisation d'un index)
- [ ] Connaître les bonnes pratiques (toujours un opérateur de mise à jour, TTL/conventions de champs cohérentes)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (sharding, replica sets, quand ne pas utiliser MongoDB)

## 10. Ressources

- [Documentation officielle MongoDB](https://www.mongodb.com/docs/manual/) — référence complète.
- [MongoDB University](https://learn.mongodb.com/) — cours gratuits officiels, y compris sur l'agrégation et le data modeling.
- [Mongoose — documentation](https://mongoosejs.com/docs/) — ODM de référence côté Node.js.
- [MongoDB Data Modeling patterns](https://www.mongodb.com/docs/manual/data-modeling/) — guide officiel sur le choix embedding vs référencement.
- [`../mysql/`](../mysql/) et [`../postgresql/`](../postgresql/) pour la comparaison directe avec un modèle relationnel.
