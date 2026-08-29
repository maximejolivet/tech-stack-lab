# Solutions — Niveau 3 (Avancé)

## Exercice 1

**Côté REST** : ajouter un paramètre d'inclusion, ex. `GET /authors?include=books`, qui retourne chaque auteur avec ses livres imbriqués dans la même réponse — une seule requête au lieu de 21.

**Côté GraphQL** : une seule requête imbriquée suffit nativement :

```graphql
query {
  authors {
    name
    books { title }
  }
}
```

## Exercice 2

Le client envoie une clé unique (`Idempotency-Key: <uuid>`) générée une fois par tentative logique de paiement (pas régénérée entre les retries du même achat). Le serveur, à la première requête reçue avec cette clé, exécute le paiement et associe le résultat à cette clé ; si une requête arrive à nouveau avec la même clé (retry après timeout), le serveur retourne directement le résultat déjà enregistré sans réexécuter le paiement, garantissant qu'un même achat n'est jamais facturé deux fois.

## Exercice 3

Il ne faut pas faire confiance au corps brut car n'importe qui connaissant l'URL du webhook pourrait envoyer une fausse notification (ex. `{"status": "paid"}` falsifié pour débloquer une commande sans paiement réel). L'authenticité se vérifie via une signature : le fournisseur du webhook signe le corps de la requête avec un secret partagé et l'envoie dans un header (ex. `X-Signature`) ; le serveur recalcule cette signature à partir du corps reçu et du même secret, et rejette la requête si les deux ne correspondent pas.

## Exercice 4

- **Fermé (closed)** : état normal, les requêtes passent vers le service tiers. Le circuit breaker compte les échecs.
- **Ouvert (open)** : dès qu'un seuil d'échecs est dépassé (ex. 100% d'erreurs 500 constatées ici), le circuit breaker "ouvre" et bloque immédiatement tous les nouveaux appels vers ce service pendant une durée définie, en retournant une erreur rapide côté application plutôt que de laisser chaque requête attendre un timeout inutilement.
- **Semi-ouvert (half-open)** : après ce délai, le circuit breaker laisse passer un nombre limité de requêtes test ; si elles réussissent, il repasse en fermé (service rétabli), sinon il repasse en ouvert pour un nouveau délai.

## Exercice 5

**Première requête (rien en cache encore)** — réponse initiale :

```
HTTP/1.1 200 OK
ETag: "a1b2c3d4"
```

**Requête suivante, avec la version en cache** :

```
GET /products/7
If-None-Match: "a1b2c3d4"
```

**Si la ressource n'a pas changé, le serveur répond sans corps** :

```
HTTP/1.1 304 Not Modified
```

Le client réutilise alors sa copie locale déjà en cache, économisant le transfert du corps de la réponse.
