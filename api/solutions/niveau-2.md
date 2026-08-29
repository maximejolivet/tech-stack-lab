# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

Avec une pagination par offset, si un commentaire est inséré en tête de liste entre le chargement de la page 1 et celui de la page 2, tous les éléments "décalent" d'une position : le client peut alors revoir un élément déjà affiché ou en manquer un. La pagination par curseur référence un point stable (ex. l'ID du dernier élément vu) : la page suivante est toujours calculée par rapport à cette position, indépendamment des insertions/suppressions survenues ailleurs dans la liste.

```json
{
  "data": [ /* commentaires */ ],
  "meta": { "next_cursor": "eyJpZCI6MTIzfQ==", "has_more": true }
}
```

## Exercice 2

C'est un changement cassant (breaking change) : tout client `/v1` existant qui fait `product.price + 10` ou affiche directement `price` comme un nombre casserait si le champ devient soudainement un objet. Il faut donc introduire `/v2/products` avec la nouvelle forme, tout en continuant à servir `/v1/products` avec l'ancienne forme (le backend peut soit maintenir deux implémentations, soit adapter/transformer la réponse `/v2` interne vers le format `/v1` pour cette route). `/v1` est ensuite dépréciée avec une date de fin de support communiquée aux clients, qui migrent vers `/v2` à leur rythme.

## Exercice 3

1. Le client envoie ses identifiants à `POST /login`.
2. Le serveur les vérifie, puis génère et signe un JWT contenant les informations nécessaires (ex. `user_id`, date d'expiration) avec une clé secrète connue uniquement du serveur.
3. Le serveur retourne ce JWT au client.
4. Le client stocke le JWT (idéalement dans un cookie `HttpOnly`) et l'envoie dans le header `Authorization: Bearer <token>` à chaque requête vers un endpoint protégé.
5. Le serveur vérifie la signature et l'expiration du token à chaque requête reçue, sans avoir besoin de consulter une session stockée côté serveur (stateless).

## Exercice 4

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 37
X-RateLimit-Reset: 1717000000
```

En cas de dépassement :

```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

## Exercice 5

GraphQL laisse le client préciser exactement les champs qu'il veut dans sa requête (`query { user(id: 1) { name, avatarUrl } }`), le serveur ne retournant que ces deux champs au lieu des 20 — plus de sur-fetching. En restant en REST, une alternative est d'ajouter un paramètre de sélection de champs (`GET /users/1?fields=name,avatar_url`), interprété côté serveur pour ne renvoyer que le sous-ensemble demandé, ou de créer un endpoint dédié plus léger pour ce cas d'usage précis (ex. `/users/1/summary`).
