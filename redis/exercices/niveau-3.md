# Exercices Redis — Niveau 3 (Avancé)

## Exercice 1 — Verrou distribué

Écris la commande qui pose un verrou distribué `lock:job:99` détenu par le worker `worker-3`, valide seulement s'il n'existe pas déjà, avec une expiration de sécurité de 30 secondes. Explique en une phrase pourquoi l'expiration est indispensable.

## Exercice 2 — Rate limiting

Conçois (commandes Redis + logique) un rate limiter qui autorise au maximum 100 requêtes par minute par adresse IP, basé sur `INCR` + `EXPIRE`. Explique la limite de cette approche simple (fenêtre fixe) par rapport à une fenêtre glissante.

## Exercice 3 — KEYS vs SCAN

Un script de maintenance exécute `KEYS session:*` sur une instance Redis en production contenant 5 millions de clés, et le service devient indisponible pendant l'exécution. Explique pourquoi, et réécris le script avec la commande appropriée pour éviter ce problème.

## Exercice 4 — Redis Cluster et hash tags

Une transaction `MULTI`/`EXEC` doit porter sur `user:1:profile` et `user:1:settings` sur un déploiement Redis Cluster, mais échoue avec une erreur indiquant que les clés ne sont pas sur le même slot. Explique le problème et corrige-le en renommant les clés avec un hash tag approprié.

## Exercice 5 — RDB vs AOF, arbitrage

Une application de type file d'attente de jobs critiques (chaque job perdu = tâche métier perdue, ex. envoi de facture) utilise Redis comme store primaire des jobs en attente. Quelle configuration de persistance recommander (RDB seul, AOF seul, ou les deux), et avec quel réglage de `fsync` pour AOF ? Justifie en 2-3 phrases le compromis performance/durabilité.
