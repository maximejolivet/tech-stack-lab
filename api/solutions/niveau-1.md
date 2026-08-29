# Solutions — Niveau 1 (Bases)

## Exercice 1

```
GET    /tasks              # lister
POST   /tasks               # créer
GET    /tasks/{id}          # détail
PATCH  /tasks/{id}          # modifier
DELETE /tasks/{id}          # supprimer
PATCH  /tasks/{id}          # marquer comme terminée : { "completed": true }
```

"Marquer comme terminée" n'est pas une action séparée mais une modification partielle de l'état de la ressource `task` — elle passe donc par `PATCH /tasks/{id}` avec le champ concerné dans le corps, pas par une route dédiée type `/tasks/{id}/complete` en `GET`.

## Exercice 2

`GET` doit rester sans effet de bord et idempotent — or `/orders/42/cancel` en `GET` annule réellement la commande, ce qui peut être déclenché accidentellement par un crawler, un pré-chargement de lien, ou un simple partage d'URL. Route corrigée : `POST /orders/42/cancel` (action métier qui ne correspond pas à un remplacement de ressource complet, donc `POST` plutôt que `PATCH`/`PUT`).

## Exercice 3

(a) `201 Created` — (b) `204 No Content` — (c) `401 Unauthorized` — (d) `422 Unprocessable Entity` — (e) `404 Not Found`.

## Exercice 4

```json
{
  "errors": {
    "email": ["Cette adresse email est déjà utilisée."],
    "password": ["Le mot de passe doit contenir au moins 8 caractères."]
  }
}
```

## Exercice 5

```json
{
  "data": [ /* 10 produits */ ],
  "meta": {
    "page": 2,
    "per_page": 10,
    "total": 47,
    "total_pages": 5
  }
}
```
