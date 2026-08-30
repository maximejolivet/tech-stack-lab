# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```javascript
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
    { $sort: { total: -1 } }
])
```

## Exercice 2

Embedder les commentaires directement dans le document `post` serait risqué ici : un post populaire peut accumuler des centaines, voire des milliers de commentaires au fil du temps, ce qui ferait grossir le document indéfiniment (jusqu'à approcher la limite de 16 Mo par document BSON) et alourdirait chaque lecture du post même quand les commentaires ne sont pas nécessaires. Mieux vaut **référencer** : une collection `comments` séparée avec un champ `postId`, interrogée indépendamment (avec pagination) quand le post est affiché.

```javascript
db.comments.insertOne({ postId: ObjectId("..."), author: "Max", text: "..." })
db.comments.find({ postId: ObjectId("...") }).limit(20)
```

## Exercice 3

```javascript
db.createCollection("users", {
    validator: {
        $jsonSchema: {
            required: ["email"],
            properties: {
                email: { bsonType: "string" },
                age: { bsonType: "number", minimum: 0 }
            }
        }
    }
})
```

## Exercice 4

```javascript
db.products.find({
    reviews: { $elemMatch: { rating: { $gte: 4 }, verified: true } }
})
```

Sans `$elemMatch`, une requête `{ "reviews.rating": { $gte: 4 }, "reviews.verified": true }` matcherait un produit dès qu'**un** avis a `rating >= 4` et qu'**un autre** avis (potentiellement différent) a `verified: true` — `$elemMatch` garantit que les deux conditions portent sur le **même** élément du tableau.

## Exercice 5

```javascript
const session = db.getMongo().startSession()
session.startTransaction()

try {
    const source = db.accounts.findOne({ _id: 1 }, { session })
    if (source.balance - 50 < 0) {
        session.abortTransaction()
        // le solde deviendrait négatif : on annule avant toute écriture
    } else {
        db.accounts.updateOne({ _id: 1 }, { $inc: { balance: -50 } }, { session })
        db.accounts.updateOne({ _id: 2 }, { $inc: { balance: 50 } }, { session })
        session.commitTransaction()
    }
} catch (e) {
    session.abortTransaction()
}
```
