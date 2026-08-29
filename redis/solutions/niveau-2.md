# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```
function getProduct(id):
    cached = redis.GET("product:" + id)
    if cached is not null:
        return deserialize(cached)

    product = db.query("SELECT * FROM products WHERE id = ?", id)
    redis.SET("product:" + id, serialize(product), EX=300)
    return product
```

## Exercice 2

```
function updateProduct(id, data):
    db.query("UPDATE products SET ... WHERE id = ?", id, data)
    redis.DEL("product:" + id)   # invalidation : le prochain getProduct repeuplera le cache à jour
```

## Exercice 3

Un abonné qui se connecte **après** la publication d'un message ne le reçoit jamais : Pub/Sub Redis ne conserve aucun historique, il ne fait que diffuser en temps réel aux abonnés déjà connectés au moment de la publication.

```bash
SUBSCRIBE notifications
PUBLISH notifications "Nouvelle commande #42"
```

## Exercice 4

`allkeys-lru` : Redis évince automatiquement les clés les moins récemment utilisées quel que soit leur TTL, ce qui est cohérent pour un cache pur où toute donnée est par nature remplaçable/recalculable. `noeviction` serait un mauvais choix ici car il refuse toute nouvelle écriture une fois la limite mémoire atteinte — l'application recevrait des erreurs d'écriture au lieu de continuer à fonctionner en dégradant simplement le taux de cache hit.

## Exercice 5

```bash
MULTI
INCRBY stock:product:1 -1
INCRBY sales:product:1 1
EXEC
```
