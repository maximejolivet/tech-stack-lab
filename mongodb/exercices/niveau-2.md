# Exercices MongoDB — Niveau 2 (Intermédiaire)

## Exercice 1 — Pipeline d'agrégation

Étant donné une collection `orders` avec les champs `customerId`, `status` et `amount`, écris un pipeline d'agrégation qui calcule, pour chaque client ayant au moins une commande `"completed"`, le montant total dépensé, trié du plus gros au plus petit acheteur.

## Exercice 2 — Embedding vs référencement

Un blog a des `posts` et des `comments`. Chaque post peut recevoir des dizaines de commentaires au fil du temps, consultés uniquement dans le contexte de leur post. Décide si les commentaires doivent être **embedded** dans le document `post` ou **référencés** dans une collection séparée `comments`, et justifie ta réponse en 3-4 lignes (pense à la taille maximale d'un document BSON — 16 Mo — et à la fréquence d'ajout de commentaires).

## Exercice 3 — Validation de schéma

Écris un `createCollection` avec un validateur JSON Schema pour une collection `users`, exigeant `email` (string, requis) et `age` (number, minimum 0, non requis).

## Exercice 4 — Requête sur tableau

Étant donné des documents `products` avec un champ `reviews` (tableau d'objets `{ rating, verified }`), écris une requête qui trouve les produits ayant au moins un avis avec `rating >= 4` ET `verified: true` dans le **même** élément du tableau (attention à ne pas se contenter de deux conditions séparées sur le tableau).

## Exercice 5 — Transaction multi-documents

Écris une transaction MongoDB qui transfère 50 (unités arbitraires) du document `{ _id: 1 }` vers le document `{ _id: 2 }` dans une collection `accounts`, avec `abortTransaction()` explicite en commentaire si le solde du compte source deviendrait négatif.
