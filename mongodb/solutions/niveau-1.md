# Solutions — Niveau 1 (Bases)

## Exercice 1

```javascript
use mon_site

db.products.insertMany([
    { name: "Clavier", price: 45, category: "informatique" },
    { name: "Souris", price: 15, category: "informatique" },
    { name: "Chaise", price: 89, category: "bureau" }
])
```

## Exercice 2

```javascript
db.products.find({ price: { $gt: 20 } }).sort({ price: -1 })

db.products.find({ category: { $in: ["informatique", "bureau"] } })
```

## Exercice 3

```javascript
db.products.updateMany(
    { category: "électronique" },
    { $mul: { price: 1.10 } }
)
```

Sans opérateur de mise à jour, `db.products.updateMany({ category: "électronique" }, { price: 50 })` serait interprété comme un **document de remplacement complet** : MongoDB écraserait chaque document matché avec `{ price: 50 }` uniquement, supprimant tous ses autres champs (`name`, `category`...).

## Exercice 4

```javascript
db.products.deleteMany({ price: { $lt: 10 } })
```

`deleteMany({})` matche un filtre vide, donc **tous** les documents de la collection : c'est l'équivalent d'un `DELETE FROM table` SQL sans `WHERE`, qui vide entièrement la collection.

## Exercice 5

```javascript
db.products.createIndex({ category: 1 })

db.products.getIndexes()
```
