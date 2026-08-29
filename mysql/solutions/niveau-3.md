# Solutions — Niveau 3 (Avancé)

## Exercice 1

`type: ALL` signifie un **scan complet de table** : MySQL parcourt les 1,2 million de lignes une par une, sans utiliser d'index, pour évaluer la condition `WHERE`. Sur une table de cette taille, c'est très coûteux en I/O et en temps de réponse. Action corrective : identifier la ou les colonnes utilisées dans le `WHERE` et créer un index dessus (`CREATE INDEX ...`), puis revérifier avec `EXPLAIN` que `type` passe à `ref`/`range`/`const`.

## Exercice 2

Quand MySQL détecte un deadlock (cycle d'attente entre transactions), il choisit automatiquement l'une des deux transactions comme "victime", annule ses effets (`ROLLBACK` implicite) et lui retourne une erreur (`Deadlock found when trying to get lock`) — l'autre transaction continue normalement. Côté applicatif, il faut intercepter cette erreur et **retenter** la transaction annulée. Pour réduire la fréquence des deadlocks, on peut imposer un ordre de verrouillage cohérent dans tout le code (ex. toujours verrouiller les comptes dans l'ordre croissant de leur `id`, jamais dans l'ordre d'appel de la fonction).

## Exercice 3

Sous `READ COMMITTED`, chaque `SELECT` au sein d'une même transaction voit les données telles que committées **au moment de ce SELECT** — deux lectures successives de la même ligne peuvent donc retourner des valeurs différentes si une autre transaction a committé un changement entre les deux (lecture non répétable). Sous `REPEATABLE READ` (défaut MySQL/InnoDB), la transaction voit un instantané cohérent établi à son démarrage : deux lectures successives de la même ligne retournent la même valeur pendant toute la durée de la transaction, même si une autre transaction committe un changement entre-temps.

## Exercice 4

La réplication MySQL est asynchrone par défaut : il existe un léger délai entre l'écriture committée sur le primaire et sa propagation vers la réplique. Si la page de profil lit immédiatement après l'écriture sur une réplique qui n'a pas encore reçu ce changement, l'utilisateur nouvellement créé n'y est pas encore visible. Solution : soit lire depuis le primaire juste après une écriture critique (lecture "read-your-writes" routée explicitement), soit utiliser une réplication semi-synchrone/attendre un accusé de réception de la réplique avant de rediriger l'utilisateur.

## Exercice 5

Dénormalisation possible : stocker un champ `total` (déjà calculé) directement sur `orders`, mis à jour à chaque ajout/modification de `order_items`, plutôt que de le recalculer par jointure/agrégation à chaque affichage de la liste. Le compromis : `total` doit désormais être maintenu manuellement en cohérence avec `order_items` (risque de divergence si une mise à jour est oubliée), ce que la normalisation stricte évitait par construction.
