# Solutions — Niveau 3 (Avancé)

## Exercice 1

```bash
SET lock:job:99 "worker-3" NX EX 30
```

L'expiration est indispensable car si `worker-3` crashe avant de libérer explicitement le verrou (`DEL`), celui-ci resterait bloqué indéfiniment pour tous les autres workers sans le TTL — l'expiration garantit une libération automatique même en cas de panne.

## Exercice 2

```bash
INCR ratelimit:ip:1.2.3.4
EXPIRE ratelimit:ip:1.2.3.4 60 NX   # ne pose le TTL que s'il n'existe pas déjà (première requête de la fenêtre)
```

Si `INCR` dépasse 100, rejeter la requête. Limite de la fenêtre fixe : un client peut envoyer 100 requêtes juste avant la fin de la fenêtre et 100 autres juste après son renouvellement, soit 200 requêtes en un temps très court concentré autour de la frontière de fenêtre — une fenêtre glissante (via un `ZSET` horodaté, purgeant les entrées trop anciennes à chaque requête) lisse ce pic en comptant réellement les requêtes des 60 dernières secondes glissantes, pas d'une fenêtre calendaire fixe.

## Exercice 3

`KEYS` parcourt **l'intégralité** de l'espace de clés en une seule opération bloquante ; sur 5 millions de clés, Redis (single-threaded pour l'exécution des commandes) reste occupé à cette seule commande pendant toute sa durée, bloquant toutes les autres requêtes du service en attente.

```bash
redis-cli --scan --pattern 'session:*'
```

`SCAN` itère par petits lots avec un curseur, sans jamais bloquer le serveur sur une seule opération longue.

## Exercice 4

Sur Redis Cluster, une opération multi-clés (transaction incluse) exige que toutes les clés impliquées résident sur le même slot de hachage — `user:1:profile` et `user:1:settings` sont hachés indépendamment et peuvent tomber sur des slots différents. La correction consiste à utiliser un hash tag (portion entre `{}`) pour forcer Redis à ne hacher que cette partie de la clé :

```
{user:1}:profile
{user:1}:settings
```

Les deux clés partagent alors le même hash tag `user:1`, donc le même slot, rendant l'opération multi-clés possible.

## Exercice 5

Pour des jobs critiques où toute perte est inacceptable, AOF est indispensable (RDB seul, par snapshots périodiques, peut perdre plusieurs minutes de jobs en cas de crash entre deux snapshots). Recommandation : activer AOF avec `appendfsync always` ou, en compromis raisonnable, `appendfsync everysec` (perte maximale théorique d'une seconde de données) — `always` garantit la durabilité maximale mais impacte le débit d'écriture à chaque commande, tandis qu'`everysec` offre un excellent compromis pour la quasi-totalité des cas d'usage critiques. RDB peut être conservé en complément pour des sauvegardes/restaurations rapides, mais ne doit pas être l'unique mécanisme de durabilité ici.
