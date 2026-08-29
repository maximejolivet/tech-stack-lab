# Exercices API Design — Niveau 2 (Intermédiaire)

## Exercice 1 — Pagination par curseur

Explique en 3-4 phrases pourquoi la pagination par curseur est plus robuste que la pagination par offset sur une liste de commentaires où de nouveaux éléments sont insérés en continu, puis écris la structure JSON d'une réponse paginée par curseur.

## Exercice 2 — Versioning

Une API `/v1/products` doit évoluer : le champ `price` (nombre, en euros) doit devenir un objet `{ "amount": number, "currency": string }`. Explique pourquoi ce changement nécessite une nouvelle version (`/v2/`), et propose une stratégie pour que les clients `/v1` existants continuent de fonctionner pendant la transition.

## Exercice 3 — Authentification JWT

Décris le flux complet, étape par étape, d'une authentification par JWT : de la requête de connexion jusqu'à l'appel d'un endpoint protégé.

## Exercice 4 — Rate limiting

Écris les headers de réponse HTTP qu'une API devrait renvoyer pour informer un client de sa limite de requêtes, du nombre de requêtes restantes, et du moment de réinitialisation — puis le code de statut et le header à renvoyer en cas de dépassement.

## Exercice 5 — REST vs GraphQL

Une application mobile affiche un profil utilisateur avec seulement son nom et son avatar, mais l'API REST existante `/users/{id}` retourne 20 champs différents. Explique en 3-4 phrases comment GraphQL résoudrait ce problème de sur-fetching, et une alternative possible en restant en REST.
