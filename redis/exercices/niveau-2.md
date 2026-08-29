# Exercices Redis — Niveau 2 (Intermédiaire)

## Exercice 1 — Cache-aside en pseudo-code

Écris en pseudo-code (langage libre) une fonction `getProduct(id)` qui vérifie d'abord Redis (`product:{id}`), et en cas d'absence, interroge une base SQL puis peuple le cache avec un TTL de 300 secondes.

## Exercice 2 — Invalidation de cache

En reprenant l'exercice 1, écris la fonction `updateProduct(id, data)` qui met à jour la base SQL puis invalide correctement la clé de cache associée.

## Exercice 3 — Pub/Sub

Décris (en 2-3 phrases) ce qui se passe si un service s'abonne au canal `notifications` **après** qu'un message y a été publié. Puis écris les commandes `SUBSCRIBE`/`PUBLISH` correspondant à un scénario simple de notification de nouvelle commande.

## Exercice 4 — Politique d'éviction

Un serveur Redis utilisé uniquement comme cache applicatif (aucune donnée critique dedans) approche sa limite mémoire configurée (`maxmemory`). Quelle `maxmemory-policy` choisir, et pourquoi `noeviction` serait un mauvais choix ici ?

## Exercice 5 — Transaction MULTI/EXEC

Écris une transaction Redis qui incrémente `stock:product:1` de -1 et `sales:product:1` de +1 de façon atomique (aucune commande d'un autre client ne doit s'intercaler entre les deux).
