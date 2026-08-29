# Redis

## 1. Introduction

Redis est une base de données en mémoire (in-memory), orientée structures de données clé-valeur, utilisée principalement comme cache mais capable de bien plus (files de messages, compteurs, classements en temps réel). Ce dossier suppose [`../mysql/`](../mysql/) ou [`../postgresql/`](../postgresql/) déjà vus et se concentre sur ce que Redis apporte **en complément** d'un SGBDR, pas en remplacement.

**À quoi sert-il ?**
- Mettre en cache des données coûteuses à recalculer ou à requêter (résultat de requête SQL, session utilisateur, réponse d'API externe).
- Structures de données spécialisées directement utilisables sans les reconstruire en code applicatif (compteurs atomiques, listes, ensembles triés pour un classement, pub/sub pour du temps réel).
- Verrous distribués, limitation de débit (rate limiting), files d'attente légères.

**Où se situe-t-il dans une architecture web ?** Couche intermédiaire entre l'application et le SGBDR principal — pas une source de vérité par défaut (sauf cas assumés), mais un accélérateur : voir son usage comme driver de cache dans [`../laravel/`](../laravel/), ou comme backend de session/queue dans [`../nodejs/`](../nodejs/).

**Avantages** : latence très faible (données en RAM), structures de données riches au-delà du simple clé-valeur, primitives atomiques utiles pour la concurrence (compteurs, verrous), simplicité opérationnelle pour un usage cache basique.
**Limites** : les données tiennent en mémoire (coût et limite de volumétrie), persistance par défaut moins garantie qu'un SGBDR transactionnel (à configurer explicitement), single-threaded sur l'exécution des commandes (une commande lente bloque les suivantes) — pas fait pour des requêtes analytiques complexes.

## 2. Prérequis

- Notions de base de SGBDR (voir [`../mysql/`](../mysql/)) pour comprendre la complémentarité cache/base de données.
- Un serveur Redis local (`redis-server`) et le client `redis-cli`, ou Docker (`docker run redis`).

## 3. Rappel des bases 🟢

### 01 - Connexion et commandes de base

**Explication** — `redis-cli` est le client en ligne de commande. Les commandes sont simples et directes.

```bash
redis-cli
SET user:1:name "Alice"
GET user:1:name
DEL user:1:name
EXISTS user:1:name
```

**Bonne pratique** : structurer les clés avec un préfixe hiérarchique cohérent (`user:1:name`, `session:abc123`) — facilite le nettoyage ciblé et la lisibilité, Redis n'impose aucune convention mais la communauté s'accorde largement sur ce format.

### 02 - Expiration (TTL)

**Explication** — Une clé peut avoir une durée de vie, après laquelle elle est automatiquement supprimée — c'est le mécanisme central de Redis en tant que cache.

```bash
SET session:abc123 "user_data" EX 3600   # expire dans 3600 secondes
TTL session:abc123                        # temps restant en secondes
PERSIST session:abc123                    # retire l'expiration
```

**Erreur fréquente** : stocker une donnée dans Redis sans TTL en pensant l'utiliser comme cache — sans expiration, la clé reste indéfiniment, ce qui transforme silencieusement le cache en source de vérité persistante non voulue, avec un risque de données obsolètes jamais rafraîchies.

### 03 - Types : String, et incréments atomiques

**Explication** — Le type `String` peut aussi stocker des nombres, avec des opérations atomiques d'incrémentation utiles pour des compteurs concurrents.

```bash
SET page:home:views 0
INCR page:home:views       # +1 atomique
INCRBY page:home:views 5   # +5 atomique
```

**Bonne pratique** : utiliser `INCR`/`INCRBY` plutôt qu'un `GET` + calcul + `SET` en code applicatif pour un compteur partagé — évite une race condition entre deux requêtes concurrentes qui liraient la même valeur avant que l'une des deux n'écrive.

### 04 - Type Hash

**Explication** — Stocke un objet à plusieurs champs sous une seule clé, plus efficace que plusieurs clés `String` séparées pour représenter une entité.

```bash
HSET user:1 name "Alice" email "alice@test.com" age 30
HGET user:1 name
HGETALL user:1
```

### 05 - Type List

**Explication** — Liste ordonnée, utilisable comme file d'attente simple (FIFO) via `LPUSH`/`RPOP`.

```bash
LPUSH queue:emails "email1" "email2"
RPOP queue:emails    # retire et retourne le plus ancien élément
LRANGE queue:emails 0 -1   # lister tous les éléments
```

### 06 - Type Set et Sorted Set

**Explication** — `Set` : collection non ordonnée d'éléments uniques (appartenance, intersection, union). `Sorted Set` (`ZSET`) : comme un Set, mais chaque élément a un score qui détermine son ordre — base d'un classement/leaderboard.

```bash
SADD tags:article:1 "sql" "postgres"
SISMEMBER tags:article:1 "sql"   # 1 (présent)

ZADD leaderboard 1500 "alice" 1200 "bob"
ZREVRANGE leaderboard 0 2 WITHSCORES   # top 3, score décroissant
```

## 4. Concepts intermédiaires 🟡

- **Redis comme cache applicatif (cache-aside)** : le pattern le plus courant — l'application vérifie d'abord Redis, et n'interroge le SGBDR qu'en cas d'absence (cache miss), en peuplant alors le cache pour la prochaine lecture.

```python
value = redis.get(f"product:{id}")
if value is None:
    value = db.query("SELECT * FROM products WHERE id = %s", id)
    redis.set(f"product:{id}", value, ex=300)  # cache 5 minutes
```

**Erreur fréquente** : oublier d'invalider (ou de mettre à jour) la clé de cache correspondante lors d'un `UPDATE` en base — l'application continue de servir une donnée obsolète jusqu'à expiration du TTL, ce qui peut être acceptable (staleness assumée) ou problématique selon le cas d'usage ; le choix doit être **explicite**, pas un oubli.

- **Pub/Sub** : diffuser un message à tous les abonnés d'un canal en temps réel — utile pour des notifications live, mais **sans garantie de livraison** (un abonné déconnecté au moment de la publication perd le message, contrairement à une vraie file de messages).

```bash
SUBSCRIBE notifications
PUBLISH notifications "Nouvelle commande #42"
```

- **Persistance : RDB vs AOF** : `RDB` (snapshot périodique complet du dataset sur disque) est rapide à restaurer mais peut perdre les dernières minutes de données ; `AOF` (Append-Only File, journal de chaque commande d'écriture) offre une durabilité plus fine (configurable jusqu'à `fsync` à chaque écriture) au prix d'un fichier plus volumineux et d'une restauration plus lente. Les deux peuvent être combinés.
- **Politiques d'éviction (`maxmemory-policy`)** : quand Redis atteint sa limite mémoire configurée, une politique détermine quelles clés sacrifier — `allkeys-lru` (les moins récemment utilisées, choix courant pour un cache pur), `volatile-lru` (idem mais uniquement parmi les clés ayant un TTL), `noeviction` (refuse les nouvelles écritures, adapté si Redis n'est pas qu'un cache).
- **Transactions Redis (`MULTI`/`EXEC`)** : regroupe plusieurs commandes en un bloc exécuté atomiquement (aucune autre commande client ne s'intercale), mais sans mécanisme de `ROLLBACK` conditionnel comme en SQL — une commande individuellement invalide dans le bloc échoue à son exécution, sans annuler les autres commandes du bloc déjà valides.

## 5. Concepts avancés 🟠🔴

- **Redis comme source de vérité (au-delà du cache)** : avec la persistance configurée correctement (AOF avec `fsync` fréquent, réplication), Redis peut servir de store primaire pour des données qui tolèrent son modèle (compteurs, sessions, files d'attente) — mais rester conscient que ce n'est pas un SGBDR transactionnel complet (pas de jointures, pas de contraintes d'intégrité référentielle).
- **Verrous distribués (`SET ... NX`)** : obtenir un verrou exclusif entre plusieurs instances applicatives via une écriture conditionnelle atomique.

```bash
SET lock:job:42 "worker-1" NX EX 30   # NX = seulement si la clé n'existe pas
```

**Bonne pratique** : toujours associer un `EX` (TTL) à un verrou distribué — sans expiration, un worker qui crashe sans libérer le verrou le laisse bloqué indéfiniment pour tous les autres.

- **Rate limiting** : combiner `INCR` + `EXPIRE` (ou l'algorithme du sliding window log via `ZSET`) pour limiter le nombre de requêtes par utilisateur/IP sur une fenêtre de temps donnée — voir [`../security/`](../security/), [`../api/`](../api/) pour le contexte plus large de protection d'API.
- **Redis Streams** : structure de données dédiée aux files de messages avec garanties plus fortes que Pub/Sub (persistance des messages, groupes de consommateurs avec accusé de réception) — plus proche d'un Kafka léger que d'un simple canal Pub/Sub.
- **Réplication et Sentinel** : un primaire réplique vers des répliques en lecture seule (comme MySQL/PostgreSQL) ; Redis Sentinel surveille le primaire et orchestre un basculement automatique (failover) vers une réplique en cas de panne, sans intervention manuelle.
- **Redis Cluster** : sharding automatique des données sur plusieurs nœuds (16384 slots de hachage répartis), permettant de dépasser la limite de mémoire d'une seule machine — complexifie certaines opérations multi-clés (une transaction ne peut porter que sur des clés du même slot, sauf usage de "hash tags" `{user:1}:profile` pour forcer leur colocation).
- **Latence et opérations bloquantes** : Redis étant single-threaded pour l'exécution des commandes, une commande coûteuse sur une grosse structure (`KEYS *` sur une base volumineuse, un `SMEMBERS` sur un Set géant) bloque toutes les autres commandes pendant son exécution — préférer `SCAN` (itération non bloquante) à `KEYS` en production, et découper les grosses structures si nécessaire.

## 6. Commandes / syntaxe à connaître

```bash
redis-cli PING                    # vérifier la connexion
redis-cli MONITOR                 # observer toutes les commandes en temps réel (debug)
redis-cli --scan --pattern 'user:*'   # itérer sur les clés sans bloquer (vs KEYS)
redis-cli INFO memory              # statistiques mémoire
redis-cli FLUSHDB                  # vider la base courante (⚠️ destructif)
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Cache et classement pour une API de produits**

- Implémenter un cache-aside pour l'endpoint `GET /products/:id` : lecture Redis d'abord, fallback SQL avec population du cache, TTL de 5 minutes.
- Invalider explicitement la clé de cache correspondante à chaque `PUT /products/:id`.
- Ajouter un compteur de vues par produit (`INCR`), affiché sur la fiche produit.
- Implémenter un classement des produits les plus vus (`ZSET`), avec un endpoint `GET /products/top` qui retourne le top 10.
- Ajouter un rate limiting simple (`INCR`+`EXPIRE`) limitant chaque IP à 100 requêtes par minute sur l'API.
- Bonus : configurer `maxmemory-policy` en `allkeys-lru` et discuter pourquoi ce choix est cohérent pour ce cas d'usage cache-only.

## Checklist

- [ ] Comprendre les fondamentaux (clé-valeur, TTL, types String/Hash/List/Set/ZSET)
- [ ] Savoir utiliser Redis comme cache applicatif (pattern cache-aside)
- [ ] Maîtriser la syntaxe principale (INCR, EXPIRE, HSET, ZADD, MULTI/EXEC)
- [ ] Comprendre les concepts importants (Pub/Sub, persistance RDB/AOF, éviction)
- [ ] Savoir debugger (`MONITOR`, `INFO`, `SCAN` plutôt que `KEYS`)
- [ ] Connaître les bonnes pratiques (TTL systématique en cache, invalidation explicite, verrous avec expiration)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Streams, Sentinel, Cluster, opérations bloquantes)

## 10. Ressources

- [Documentation officielle Redis](https://redis.io/docs/latest/) — référence complète des commandes et structures.
- [Redis University](https://university.redis.com/) — cours gratuits officiels.
- Il n'existe pas de roadmap.sh dédiée à Redis à ce jour ; voir [roadmap.sh — Backend](https://roadmap.sh/backend) pour le contexte plus large, et [`../mysql/`](../mysql/)/[`../postgresql/`](../postgresql/) pour la complémentarité cache/SGBDR.
