# System Design

## 1. Introduction

Le System Design est la discipline qui consiste à concevoir l'architecture globale d'une application à l'échelle : comment répartir la charge entre plusieurs serveurs, où placer le cache, comment faire évoluer la base de données, comment les services communiquent entre eux. Ce dossier suppose acquis les bases backend (voir [`../php/`](../php/), [`../nodejs/`](../nodejs/)), les bases de données relationnelles (voir [`../mysql/`](../mysql/), [`../postgresql/`](../postgresql/)), le cache (voir [`../redis/`](../redis/)), la conteneurisation (voir [`../docker/`](../docker/), [`../kubernetes/`](../kubernetes/)) et la conception d'API (voir [`../api/`](../api/)) — il se concentre sur **comment assembler ces briques à l'échelle**, pas sur chacune individuellement.

**À quoi ça sert ?**
- Concevoir une architecture qui tient la charge quand le nombre d'utilisateurs ou le volume de données grandit de 10x, 100x, 1000x.
- Faire des choix de compromis (cohérence vs disponibilité, simplicité vs scalabilité) en connaissance de cause plutôt que par défaut.
- Réussir les entretiens "system design", très présents dans le recrutement backend/senior.

**Où ça se situe ?** C'est la couche la plus haute de l'architecture logicielle : elle orchestre les choix technos vus dans les autres dossiers (langage, framework, base de données, cache, infra) en fonction de contraintes réelles (charge, latence, disponibilité, budget).

**Avantages** : une architecture pensée en amont évite des refontes coûteuses, un vocabulaire commun (load balancer, cache-aside, sharding) facilite la communication entre équipes et avec des architectes/seniors.
**Limites** : sur-architecturer un petit projet (introduire microservices, message queue, sharding avant d'en avoir besoin) ajoute de la complexité opérationnelle sans bénéfice réel — le System Design s'apprend aussi en reconnaissant *quand ne pas* appliquer ces patterns.

## 2. Prérequis

- Bases backend et bases de données relationnelles (voir [`../php/`](../php/), [`../nodejs/`](../nodejs/), [`../mysql/`](../mysql/), [`../postgresql/`](../postgresql/)).
- Notions de cache (voir [`../redis/`](../redis/)) et de conception d'API (voir [`../api/`](../api/)).
- Bases de conteneurisation/orchestration utiles pour comprendre le déploiement à l'échelle (voir [`../docker/`](../docker/), [`../kubernetes/`](../kubernetes/)).

## 3. Rappel des bases 🟢

### 01 - Architecture client-serveur

**Explication** — Modèle de base : un client (navigateur, app mobile) envoie une requête à un serveur qui la traite et renvoie une réponse. Toute l'architecture web repose sur cette brique, étendue avec des couches intermédiaires (load balancer, cache, CDN) à mesure que la charge augmente.

```text
Client (navigateur) --HTTP--> Serveur applicatif --SQL--> Base de données
```

**Bonne pratique** : toujours commencer par comprendre le chemin d'une requête de bout en bout avant d'introduire une optimisation — beaucoup de problèmes de performance viennent d'un maillon simple mal compris plutôt que d'un manque d'architecture sophistiquée.

### 02 - Scalabilité verticale vs horizontale

**Explication** — **Scalabilité verticale (scale up)** : augmenter la puissance d'une seule machine (plus de CPU/RAM). **Scalabilité horizontale (scale out)** : ajouter plusieurs machines qui se répartissent la charge.

**Erreur fréquente** : penser que la scalabilité verticale est "plus simple donc suffisante" indéfiniment — elle a une limite physique (la machine la plus puissante disponible) et un point de défaillance unique (si cette machine tombe, tout le service tombe).

**Bonne pratique** : la scalabilité verticale est un bon premier réflexe pour un projet naissant (simplicité), la scalabilité horizontale devient nécessaire dès que la disponibilité (tolérance de panne) ou une charge dépassant une seule machine deviennent des exigences réelles.

### 03 - Load balancer (répartiteur de charge)

**Explication** — Répartit les requêtes entrantes entre plusieurs serveurs applicatifs identiques, permettant la scalabilité horizontale et la tolérance de panne (si un serveur tombe, le load balancer arrête de lui envoyer du trafic).

```text
Clients --> Load Balancer --> [Serveur A, Serveur B, Serveur C]
```

**Stratégies courantes** : round-robin (à tour de rôle), least connections (vers le serveur le moins chargé), par hash (même client → même serveur, utile pour des sessions en mémoire locale).

**Erreur fréquente** : garder l'état de session en mémoire locale sur chaque serveur applicatif derrière un load balancer round-robin — un utilisateur peut atterrir sur un serveur différent à chaque requête et perdre sa session. Externaliser la session (voir [`../redis/`](../redis/)) ou utiliser le hash par client (sticky sessions), en sachant que les sticky sessions réintroduisent une forme de couplage client-serveur.

### 04 - Cache : principe de base

**Explication** — Stocker temporairement le résultat d'une opération coûteuse (requête base de données, appel API externe, calcul lourd) pour éviter de la refaire à chaque requête identique.

**Pattern cache-aside (le plus courant)** : l'application vérifie d'abord le cache ; si absent (cache miss), elle interroge la source de vérité (base de données) puis remplit le cache pour la prochaine fois.

```text
1. Lire le cache (ex. Redis) pour la clé "user:42"
2. Si présent (cache hit) -> retourner directement
3. Si absent (cache miss) -> lire en base, écrire le résultat dans le cache, puis retourner
```

**Erreur fréquente** : oublier d'invalider (ou de mettre à jour) le cache après une écriture en base — l'application sert alors une donnée périmée ("stale") sans erreur visible.

### 05 - Réplication de base de données

**Explication** — Une base **primaire (primary/master)** reçoit les écritures, une ou plusieurs bases **répliques (replicas)** copient ses données en (quasi) temps réel et absorbent les lectures — répartit la charge de lecture, qui domine largement l'écriture dans la plupart des applications web.

```text
Écritures --> Base primaire --réplication--> Réplique 1, Réplique 2 (lectures)
```

**Erreur fréquente** : supposer qu'une lecture juste après une écriture voit toujours la donnée à jour — la réplication est généralement asynchrone, avec un délai (replication lag) pendant lequel une réplique peut retourner une version légèrement périmée.

### 06 - Sharding (partitionnement horizontal)

**Explication** — Répartir les données d'une même table entre plusieurs bases (shards) selon une clé de partitionnement (ex. `user_id % nombre_de_shards`), pour dépasser la capacité d'une seule machine en écriture — contrairement à la réplication qui duplique les mêmes données pour la lecture.

**Bonne pratique** : le sharding introduit une complexité significative (les requêtes qui traversent plusieurs shards, les jointures entre shards) — à n'envisager qu'une fois la verticale et la réplication en lecture épuisées, jamais comme premier réflexe.

### 07 - CDN (Content Delivery Network)

**Explication** — Réseau de serveurs répartis géographiquement qui mettent en cache des contenus statiques (images, CSS, JS, vidéos) au plus proche de l'utilisateur final, réduisant la latence et déchargeant le serveur d'origine.

**Cas d'usage** : assets statiques d'un site web, mais aussi de plus en plus des réponses API cacheables — voir [`../accessibility-performance/`](../accessibility-performance/) pour l'impact sur les Core Web Vitals.

## 4. Concepts intermédiaires 🟡

- **Files de messages (message queues)** : découplent un producteur et un ou plusieurs consommateurs — le producteur publie un message sans attendre qu'il soit traité, un ou plusieurs workers le consomment de façon asynchrone. Kafka (flux d'événements à très haut débit, conservation configurable des messages, multi-consommateurs) et RabbitMQ (files de tâches classiques, routage flexible) sont les références.

```text
Service Commande --publie--> Queue "commande.creee" --consomme--> Service Email, Service Facturation
```

- **CAP theorem** : dans un système distribué, en cas de partition réseau (P, qui finit toujours par arriver), il faut choisir entre **Cohérence (C)** — toutes les répliques voient la même donnée au même instant — et **Disponibilité (A)** — le système continue de répondre même si certaines répliques sont injoignables. On ne peut garantir C et A simultanément pendant une partition.
- **Cohérence à terme (eventual consistency)** : accepter qu'une donnée mette un court instant à se propager à toutes les répliques, en échange d'une meilleure disponibilité et de meilleures performances — modèle courant des bases NoSQL distribuées et des systèmes de cache.
- **Rate limiting** : limiter le nombre de requêtes qu'un client peut faire sur une période donnée, pour protéger le système d'un abus (volontaire ou non) — algorithmes courants : token bucket, sliding window. Voir [`../api/`](../api/) et [`../security/`](../security/) pour l'implémentation côté API.
- **Circuit breaker** : quand un service dépendant échoue de façon répétée, arrêter temporairement de l'appeler (au lieu de saturer les threads/connexions en attente d'un timeout) et retourner une réponse dégradée immédiate — protège le système appelant d'une défaillance en cascade.
- **Monolithe vs microservices** : un monolithe (une seule base de code déployée comme un tout) est plus simple à développer et déployer au début ; les microservices (services indépendants, déployés séparément, communiquant par API/queue) apportent une scalabilité et une isolation d'équipe/de panne plus fines, au prix d'une complexité opérationnelle (réseau, observabilité, cohérence des données entre services) largement supérieure. Ne pas migrer vers les microservices sans un besoin réel d'isolation ou de scalabilité indépendante.
- **Étude de cas — raccourcisseur d'URL** : un service qui transforme une URL longue en URL courte (`bit.ly/abc123`). Points clés à raisonner : génération d'un identifiant court unique (compteur encodé en base62, ou hash), stockage clé-valeur (fort ratio lecture/écriture → cache agressif devant la base), redirection HTTP 301/302 (302 pour garder la main sur les statistiques de clics), et estimation de charge (nombre d'URL créées/jour × durée de rétention).

## 5. Concepts avancés 🟠🔴

- **Étude de cas — fil d'actualité (feed)** : afficher les publications récentes des comptes suivis par un utilisateur, à l'échelle. Deux stratégies opposées : **push (fan-out à l'écriture)** — à chaque publication, insérer l'entrée dans le feed précalculé de chaque follower (rapide à lire, coûteux à écrire pour un compte à très nombreux followers) ; **pull (fan-out à la lecture)** — au moment de consulter son feed, agréger en temps réel les publications des comptes suivis (coûteux à lire, pas de coût d'écriture supplémentaire). En pratique, une approche hybride : push pour la majorité des utilisateurs, pull pour les comptes à très large audience (célébrités).
- **Idempotence** : concevoir une opération pour qu'elle produise le même résultat si elle est exécutée plusieurs fois (ex. avec une clé d'idempotence côté client) — indispensable dès qu'un réseau ou une queue peut redélivrer un même message plus d'une fois.
- **Observabilité à l'échelle** : logs centralisés, métriques (latence p50/p95/p99, taux d'erreur), tracing distribué (suivre une requête à travers plusieurs services) — sans ça, diagnostiquer un incident dans une architecture distribuée devient très difficile.
- **Bases de données : SQL vs NoSQL à l'échelle** : les bases relationnelles (voir [`../mysql/`](../mysql/), [`../postgresql/`](../postgresql/)) offrent des transactions ACID fortes mais scalent horizontalement en écriture avec plus de friction (sharding) ; les bases NoSQL (document, clé-valeur, colonne large) sacrifient souvent une partie de la cohérence ou des jointures pour une scalabilité horizontale plus native — le choix dépend du besoin réel de cohérence forte vs de débit d'écriture massif, pas d'une mode technologique.
- **Dégradation gracieuse (graceful degradation)** : concevoir le système pour qu'une panne partielle (ex. le service de recommandations est indisponible) désactive une fonctionnalité secondaire plutôt que de faire tomber toute l'application — lié au circuit breaker.
- **Coût et complexité comme contrainte de conception** : le System Design n'est pas un exercice académique sans limite — chaque brique ajoutée (queue, cache, réplication, sharding) a un coût d'infrastructure et un coût opérationnel (supervision, expertise nécessaire). Une bonne conception résout le problème réel avec la complexité minimale suffisante, pas la plus impressionnante.

## 6. Commandes / syntaxe à connaître

Le System Design est avant tout un exercice de conception et de communication (schémas, compromis argumentés), pas une syntaxe à mémoriser. Points de repère chiffrés utiles en entretien/estimation :

```text
1 million de requêtes/jour  ≈ 12 req/s en moyenne (à multiplier par un facteur de pic, ex. x5-x10)
1 caractère                 = 1 octet (ASCII) — utile pour estimer un volume de stockage texte
Latence mémoire (RAM)       ≈ 100 ns
Latence SSD                 ≈ 100 µs
Aller-retour réseau (même région) ≈ 1 ms
Aller-retour réseau (intercontinental) ≈ 100-150 ms
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Concevoir (sur schéma + document) l'architecture d'un raccourcisseur d'URL à l'échelle**

Produire un document d'architecture (schéma + explications écrites) couvrant :
- Le schéma de données (table/collection URLs, clé de partitionnement éventuelle).
- La génération de l'identifiant court (algorithme choisi et pourquoi) et l'estimation de volumétrie (nombre d'URL/jour, taille de stockage sur 1 an).
- Le placement du cache (quelle donnée, quelle politique d'expiration) et son impact sur le ratio lecture/écriture.
- La stratégie de scalabilité (réplication en lecture ? sharding ? à partir de quel volume ?).
- Le choix HTTP 301 vs 302 pour la redirection, argumenté.
- Bonus : ajouter un système de statistiques de clics (compteur par URL) sans ralentir le chemin critique de la redirection — quelle brique (queue ? écriture asynchrone ?) utiliser.

## Checklist

- [ ] Comprendre les fondamentaux (client-serveur, scalabilité verticale/horizontale, load balancer)
- [ ] Savoir expliquer le pattern cache-aside et ses pièges (invalidation, cache stale)
- [ ] Maîtriser le vocabulaire principal (réplication, sharding, CDN, CAP theorem)
- [ ] Comprendre les concepts importants (message queues, rate limiting, circuit breaker, monolithe vs microservices)
- [ ] Savoir raisonner un cas concret à l'oral (raccourcisseur d'URL, fil d'actualité)
- [ ] Connaître les bonnes pratiques (ne pas sur-architecturer, dégradation gracieuse, idempotence)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet (document d'architecture)
- [ ] Comprendre les notions avancées (observabilité distribuée, SQL vs NoSQL à l'échelle, fan-out push/pull)

## 10. Ressources

- [roadmap.sh — System Design](https://roadmap.sh/system-design) — roadmap dédiée, bon point d'entrée structuré.
- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer) — référence communautaire très complète, avec des études de cas détaillées.
- [`../api/`](../api/), [`../redis/`](../redis/), [`../docker/`](../docker/), [`../kubernetes/`](../kubernetes/) — briques techniques mobilisées par les architectures décrites ici.
