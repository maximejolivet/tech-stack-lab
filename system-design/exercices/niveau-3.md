# Exercices System Design — Niveau 3 (Avancé)

## Exercice 1 — Fil d'actualité : push vs pull

Pour un réseau social où la plupart des comptes ont moins de 500 followers mais quelques comptes en ont 50 millions, conçois une stratégie hybride de génération de fil d'actualité (fan-out push/pull). Justifie le seuil que tu choisirais pour basculer d'une stratégie à l'autre.

## Exercice 2 — Sharding et requêtes cross-shard

Une table `commandes` est shardée par `user_id % 16`. Un rapport doit calculer le chiffre d'affaires total du site sur le dernier mois, toutes commandes confondues. Explique pourquoi cette requête devient problématique avec le sharding, et propose une approche pour la rendre praticable (sans revenir à une base unique).

## Exercice 3 — Idempotence

Un client mobile envoie `POST /paiements` mais ne reçoit pas de réponse (timeout réseau) alors que le paiement a en réalité été traité côté serveur. Le client retente automatiquement la requête. Explique comment une clé d'idempotence évite un double paiement, et à quel niveau (client, serveur, base de données) elle doit être vérifiée.

## Exercice 4 — SQL vs NoSQL pour un cas donné

Pour un système de messagerie instantanée stockant l'historique de conversations (écritures très fréquentes, lectures par conversation uniquement, pas de jointures complexes nécessaires), argumente en 4-5 phrases le choix entre une base relationnelle et une base NoSQL adaptée, en citant les critères qui pèsent le plus dans cette décision.

## Exercice 5 — Observabilité distribuée

Une requête utilisateur traverse 4 microservices avant de recevoir une réponse, et la latence globale a doublé cette semaine sans qu'aucun service ne rapporte d'erreur. Explique comment le tracing distribué permettrait d'identifier quel service (ou quel appel réseau entre services) est responsable du ralentissement, là où des logs isolés par service ne le permettraient pas facilement.
