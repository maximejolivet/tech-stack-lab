# Solutions — Niveau 3 (Avancé)

## Exercice 1

`createdAt` seul concentre systématiquement les écritures récentes sur le shard qui héberge la plage de temps "maintenant" : à tout instant, un seul shard reçoit la quasi-totalité du trafic d'écriture, tandis que les autres restent sous-utilisés — c'est un "hot shard" typique d'une clé monotone croissante. Une clé composée `{ tenantId: 1, createdAt: 1 }` répartit d'abord les documents par client (une dimension avec de nombreuses valeurs distinctes et un trafic globalement équilibré entre tenants), tout en conservant un ordre temporel utile à l'intérieur de chaque tenant.

## Exercice 2

`secondaryPreferred` peut router la lecture vers un secondaire qui n'a pas encore reçu la réplication de l'écriture récente (réplication asynchrone par défaut, exactement le même phénomène que la réplication MySQL vue dans [`../../mysql/exercices/niveau-3.md`](../../mysql/exercices/niveau-3.md)) : le document existe bien sur le primaire mais pas encore sur le secondaire interrogé. Solution : soit lire depuis le primaire juste après une écriture critique (`readPreference: "primary"` pour cette requête précise), soit utiliser un *read concern* `"majority"` combiné à un *write concern* adapté pour garantir la visibilité de l'écriture avant la lecture suivante.

## Exercice 3

```javascript
db.orders.createIndex({ status: 1, createdAt: -1 })
```

L'ordre des champs compte car un index composé est une structure triée hiérarchiquement : `{ status: 1, createdAt: -1 }` regroupe d'abord par `status` puis trie par `createdAt` à l'intérieur de chaque groupe — exactement l'ordre du pipeline (`$match` sur `status` puis `$sort` sur `createdAt`), ce qui permet à MongoDB de satisfaire le filtre ET le tri avec le même index sans étape de tri séparée en mémoire. L'ordre inverse (`createdAt` puis `status`) ne permettrait pas d'exploiter aussi efficacement le filtre sur `status`.

## Exercice 4

`COLLSCAN` signifie un **scan complet de collection** : MongoDB examine les 3 millions de documents un par un pour n'en retourner que 42, exactement l'équivalent du `type: ALL` déjà vu côté MySQL — coûteux en I/O sur une collection de cette taille, surtout si cette requête est fréquente. Action corrective : identifier le(s) champ(s) du filtre `find` et créer un index dessus, puis revérifier avec `.explain()` que `stage` passe à `IXSCAN` (utilisation d'un index) et que `totalDocsExamined` se rapproche de `nReturned`.

## Exercice 5

Un virement comptable impliquant plusieurs tables liées (comptes, écritures, journaux d'audit) a besoin de contraintes d'intégrité référentielle **vérifiées par la base elle-même** (clé étrangère qui interdit une écriture orpheline), de jointures natives efficaces pour reconstituer l'état consolidé d'un compte, et de garanties transactionnelles ACID matures et centrales à l'usage courant du moteur — exactement ce que MySQL/PostgreSQL offrent nativement depuis des décennies. MongoDB propose des transactions multi-documents depuis la version 4.0, mais elles restent plus coûteuses en performance et l'intégrité référentielle doit être maintenue par l'application plutôt que garantie structurellement par la base : sur un domaine où une incohérence a un coût réglementaire ou financier direct, s'appuyer sur des contraintes vérifiées par le SGBDR reste le choix le plus sûr, quitte à utiliser MongoDB ailleurs dans le même système (ex. logs applicatifs, contenu éditorial) où cette rigueur n'est pas nécessaire.
