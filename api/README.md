# API Design

## 1. Introduction

Ce dossier est une **synthèse transverse** : il ne présente pas un nouveau langage ou framework, mais les principes de conception d'API (REST et GraphQL) qui s'appliquent par-dessus les frameworks backend déjà couverts ([`../laravel/`](../laravel/), [`../symfony/`](../symfony/), [`../nodejs/`](../nodejs/), [`../django/`](../django/), [`../springboot/`](../springboot/)) — tous exposent déjà des endpoints, ce dossier explique comment les concevoir correctement.

**À quoi sert-elle ?**
- Exposer les données et fonctionnalités d'un backend à des clients variés (frontend web, application mobile, autre service backend) via un contrat stable et prévisible.
- Découpler l'évolution du backend de celle des clients qui le consomment, grâce à un contrat explicite (versioning, documentation).

**Où se situe-t-elle dans une architecture web ?** À la frontière entre backend et consommateurs — une bonne API isole les clients des détails d'implémentation internes (structure de la base de données, choix technologiques du backend).

**Avantages** : un contrat bien conçu réduit les frictions d'intégration, facilite les tests automatisés, permet à plusieurs équipes/clients de consommer le même backend sans coordination constante.
**Limites** : une API mal versionnée casse ses clients à chaque évolution ; une API mal pensée dès le départ (ressources mal découpées, pas de pagination) devient très coûteuse à corriger une fois des clients en production dessus.

## 2. Prérequis

- Un framework backend pratiqué pour exposer des routes/contrôleurs (voir [`../laravel/`](../laravel/), [`../nodejs/`](../nodejs/), [`../django/`](../django/)...).
- Bases HTTP : verbes, codes de statut, headers (voir [`../html/`](../html/)).
- Notions de sécurité applicative — voir [`../security/`](../security/), en particulier authentification/autorisation.

## 3. Rappel des bases 🟢

### 01 - Qu'est-ce qui rend une API "RESTful" ?

**Explication** — REST (Representational State Transfer) est un style architectural, pas un protocole : une API RESTful expose des **ressources** (noms, ex. `/users`) manipulées via les verbes HTTP standards, de façon **stateless** (chaque requête contient tout le nécessaire, le serveur ne maintient pas de session applicative entre deux appels).

**Bonne pratique** : nommer les ressources avec des noms au pluriel (`/orders`, pas `/getOrders` ni `/order`) — l'action est portée par le verbe HTTP, pas par l'URL.

### 02 - Verbes HTTP et sémantique

**Explication** — Chaque verbe a une sémantique attendue : `GET` (lecture, sans effet de bord), `POST` (création), `PUT` (remplacement complet d'une ressource), `PATCH` (modification partielle), `DELETE` (suppression).

```
GET    /orders          # lister
GET    /orders/42       # détail
POST   /orders          # créer
PATCH  /orders/42        # modifier partiellement
DELETE /orders/42        # supprimer
```

**Erreur fréquente** : utiliser `GET` pour une action qui modifie une donnée (ex. `/orders/42/cancel` en GET, déclenché par un simple lien) — un `GET` doit rester **idempotent et sans effet de bord**, sans quoi un crawler ou un pré-chargement de navigateur peut déclencher l'action par accident.

### 03 - Codes de statut HTTP

**Explication** — Le code de statut porte le sens du résultat, indépendamment du corps de la réponse : `200 OK` (succès), `201 Created` (ressource créée, avec un header `Location`), `204 No Content` (succès sans corps, ex. après un `DELETE`), `400 Bad Request` (requête mal formée), `401 Unauthorized` (non authentifié), `403 Forbidden` (authentifié mais non autorisé), `404 Not Found`, `422 Unprocessable Entity` (validation échouée), `500 Internal Server Error`.

**Erreur fréquente** : retourner `200 OK` avec un champ `{"success": false}` dans le corps — cela oblige chaque client à parser le corps pour savoir si la requête a réussi, alors que le code de statut est fait pour ça et est directement exploitable par tout outillage HTTP générique.

### 04 - Forme d'une requête/réponse

**Explication** — Le format d'échange standard est JSON, avec le header `Content-Type: application/json`. Une réponse d'erreur doit rester structurée et prévisible.

```json
// Réponse 201 Created
{ "id": 42, "status": "pending", "total": 59.90 }

// Réponse 422 Unprocessable Entity
{ "errors": { "email": ["Le format de l'email est invalide."] } }
```

**Bonne pratique** : garder une structure d'erreur cohérente sur **toute** l'API, pas une par endpoint — cela permet aux clients d'écrire une gestion d'erreur générique une seule fois.

### 05 - Pagination

**Explication** — Deux approches principales : la **pagination par offset** (`?page=2&per_page=20`, simple mais incohérente si des lignes sont insérées/supprimées entre deux pages) et la **pagination par curseur** (`?cursor=eyJpZCI6NDJ9`, référence un point stable dans la liste, plus robuste sur des données qui changent, standard sur les grandes API comme Stripe ou GitHub).

```json
{
  "data": [ /* ... */ ],
  "meta": { "next_cursor": "eyJpZCI6NjJ9", "has_more": true }
}
```

**Bonne pratique** : ne jamais renvoyer une liste non paginée sur une ressource dont le volume peut croître sans limite.

### 06 - Versioning

**Explication** — Une API évolue ; le versioning permet de faire coexister une ancienne et une nouvelle version le temps que les clients migrent. Deux approches courantes : dans l'URL (`/v1/orders`, simple et visible) ou dans un header (`Accept: application/vnd.monapi.v2+json`, plus "propre" REST-orthodoxe mais moins discoverable).

**Bonne pratique** : ne créer une nouvelle version majeure que pour un changement **cassant** (breaking change) — ajouter un champ optionnel à une réponse existante n'en nécessite pas.

## 4. Concepts intermédiaires 🟡

- **GraphQL vs REST** : GraphQL expose un unique endpoint où le client précise exactement les champs voulus dans sa requête (`query { user(id: 1) { name, orders { total } } }`), évitant le sur- ou sous-fetching typique d'une API REST figée. En contrepartie, il perd une partie des bénéfices HTTP natifs (cache HTTP standard par URL, codes de statut sémantiques) et demande un outillage dédié côté serveur (résolveurs, schéma).
- **Authentification par token — JWT** : le client envoie un token signé dans le header `Authorization: Bearer <token>` à chaque requête ; le serveur le vérifie sans état partagé (voir les pièges associés dans [`../security/`](../security/)).
- **OAuth2** : protocole standard pour déléguer un accès sans partager de mot de passe (ex. "Se connecter avec Google") — le flow *Authorization Code* échange un code temporaire contre un token via un aller-retour serveur-à-serveur, le flow *Client Credentials* sert à une authentification service-à-service sans utilisateur.
- **Rate limiting / throttling** : limiter le nombre de requêtes par client (souvent par clé d'API) dans une fenêtre de temps, communiqué via des headers standards (`X-RateLimit-Limit`, `X-RateLimit-Remaining`) et un code `429 Too Many Requests` en cas de dépassement.
- **HATEOAS** : niveau REST le plus strict, où la réponse inclut les liens vers les actions possibles depuis la ressource courante (ex. `"links": {"cancel": "/orders/42/cancel"}`), rendant l'API "auto-descriptive". Peu appliqué en pratique dans la majorité des API grand public, mais utile à connaître conceptuellement.
- **Documentation OpenAPI/Swagger** : décrire le contrat d'une API (routes, schémas de requête/réponse, codes d'erreur) dans un fichier YAML/JSON standardisé, permettant de générer automatiquement une documentation interactive et des clients typés.
- **Idempotency** : une requête `PUT`/`DELETE` répétée plusieurs fois doit produire le même état final qu'une seule exécution — important pour les retries réseau automatiques côté client, souvent renforcé côté écriture par une clé d'idempotence (`Idempotency-Key` header) sur les `POST` de paiement par exemple.

## 5. Concepts avancés 🟠🔴

- **API Gateway** : point d'entrée unique devant plusieurs services backend, qui centralise l'authentification, le rate limiting, le routing et parfois l'agrégation de plusieurs appels internes en une seule réponse pour le client — utile en architecture microservices.
- **Cache HTTP** : headers `ETag`/`Last-Modified` combinés à `If-None-Match`/`If-Modified-Since` permettent au client de ne re-télécharger une ressource que si elle a changé (`304 Not Modified`), réduisant charge serveur et bande passante.
- **Webhooks** : au lieu que le client interroge (*poll*) l'API pour un changement d'état, le serveur notifie activement le client via une requête HTTP sortante vers une URL qu'il a enregistrée — nécessite une vérification de signature côté client pour garantir l'authenticité de l'appel entrant.
- **Résilience côté client d'API** : *retry* avec *backoff exponentiel* sur les erreurs transitoires (5xx, timeout), *circuit breaker* pour arrêter d'appeler un service qui échoue systématiquement plutôt que de saturer la file d'attente, timeouts explicites sur chaque appel.
- **N+1 côté API** : une API qui expose une liste puis nécessite un appel séparé par élément pour ses détails associés force le client à multiplier les requêtes — GraphQL adresse ce problème nativement (une seule requête imbriquée), une API REST peut l'adresser via des paramètres d'*inclusion* (`?include=customer,items`).
- **Tests de contrat (contract testing)** : au-delà des tests unitaires classiques (voir [`../testing/`](../testing/)), vérifier automatiquement qu'une réponse d'API respecte toujours son schéma documenté (OpenAPI) — évite qu'un changement de backend casse silencieusement un client sans que personne ne le remarque avant la production.

## 6. Commandes / syntaxe à connaître

```bash
curl -X POST https://api.exemple.com/orders \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"product_id": 7, "quantity": 2}'
```

```yaml
# extrait OpenAPI
paths:
  /orders/{id}:
    get:
      responses:
        '200':
          description: Commande trouvée
        '404':
          description: Commande introuvable
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**API REST de gestion de bibliothèque**

Concevoir (sur papier/markdown, sans forcément l'implémenter dans un framework précis — ou en s'appuyant sur un framework déjà vu comme [`../laravel/`](../laravel/) ou [`../nodejs/`](../nodejs/)) une API REST couvrant :
- Ressources `books` et `authors`, avec les routes CRUD standards et la sémantique de verbes/codes de statut correcte.
- Pagination par curseur sur `GET /books`.
- Un système de versioning dans l'URL (`/v1/...`).
- Une authentification par token Bearer, avec réponses `401`/`403` correctement différenciées.
- Une spécification OpenAPI minimale décrivant au moins deux endpoints.
- Bonus : ajouter un endpoint qui illustre le problème du N+1 (`GET /authors/{id}/books`) puis proposer un paramètre `?include=books` pour l'éviter en une seule requête.

## Checklist

- [ ] Comprendre les fondamentaux (verbes HTTP, codes de statut, forme requête/réponse)
- [ ] Savoir concevoir des routes RESTful cohérentes pour une ressource donnée
- [ ] Maîtriser la pagination (offset vs curseur) et le versioning
- [ ] Comprendre les concepts importants (JWT, OAuth2, rate limiting, OpenAPI)
- [ ] Savoir différencier REST et GraphQL et choisir selon le contexte
- [ ] Connaître les bonnes pratiques (idempotence, structure d'erreur cohérente, cache HTTP)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (API Gateway, webhooks, résilience, contract testing)

## 10. Ressources

- [roadmap.sh — API Design](https://roadmap.sh/api-design) — vue d'ensemble structurée de la conception d'API.
- [MDN — HTTP](https://developer.mozilla.org/fr/docs/Web/HTTP) — référence des verbes, headers et codes de statut.
- [OpenAPI Specification](https://swagger.io/specification/) — référence du format de documentation d'API.
- [GraphQL — Documentation officielle](https://graphql.org/learn/) — bases du langage de requête et du schéma.
- Voir aussi [`../security/`](../security/) pour l'authentification/autorisation, et [`../testing/`](../testing/) pour les tests d'API et de contrat.
