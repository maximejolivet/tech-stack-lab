# Solutions — Niveau 3 (Avancé)

## Exercice 1

Stratégie hybride : pour les comptes suivis ayant un nombre de followers "normal" (ex. moins de 100 000), utiliser le **push** (fan-out à l'écriture) — à chaque publication, insérer l'entrée dans le feed précalculé de chaque follower, ce qui rend la lecture du feed quasi instantanée pour l'immense majorité des utilisateurs. Pour les comptes à très large audience (au-delà du seuil choisi, ex. 100 000+ followers), utiliser le **pull** (fan-out à la lecture) — ne pas écrire dans des millions de feeds à chaque publication (coût d'écriture prohibitif et lenteur), mais fusionner leurs publications à la volée au moment de la lecture du feed de chaque follower.

Le seuil se justifie par le coût d'écriture : publier une fois pour un compte à 50 millions de followers impliquerait 50 millions d'écritures individuelles si on appliquait le push sans exception, alors que ce même compte représente une fraction négligeable des publications totales du réseau — le pull limite le surcoût au moment de la lecture, uniquement pour les followers de ces comptes spécifiques.

## Exercice 2

**Problème** : la requête doit interroger les 16 shards séparément (chacun ne contenant qu'une partie des commandes), puis agréger les résultats côté application — ce qui est plus lent, plus complexe à écrire, et ne bénéficie pas des optimisations d'agrégation natives d'une seule base.

**Approche praticable** : maintenir une table (ou un entrepôt de données séparé, ex. data warehouse alimenté par un pipeline ETL/CDC) qui agrège déjà les métriques de chiffre d'affaires par période, mise à jour de façon asynchrone à partir des écritures sur chaque shard — le rapport lit cette table pré-agrégée au lieu d'interroger les 16 shards en temps réel à chaque demande. Pour un besoin ponctuel non temps-réel, une requête fan-out (interroger les 16 shards en parallèle puis sommer côté application) reste une option acceptable, tant que ce n'est pas un chemin de requête fréquent ou critique en latence.

## Exercice 3

Une clé d'idempotence (générée côté client, ex. un UUID unique par tentative de paiement logique, réutilisé lors des retries de la même intention de paiement) est envoyée avec la requête `POST /paiements`. Côté serveur, avant de traiter le paiement, on vérifie si cette clé a déjà été traitée (stockée en base ou en cache avec le résultat de la première exécution) : si oui, on retourne directement le résultat déjà obtenu sans retraiter le paiement ; si non, on traite le paiement et on enregistre la clé avec son résultat. La vérification doit se faire côté **serveur** (au niveau de la base de données, avec une contrainte d'unicité sur la clé pour éviter une race condition si deux requêtes identiques arrivent presque simultanément) — faire confiance au client seul ne protège pas contre un vrai double envoi réseau.

## Exercice 4

Une base NoSQL orientée document ou clé-valeur convient mieux ici : les écritures sont très fréquentes (chaque message) et doivent être rapides et scalables horizontalement sans la friction du sharding relationnel ; les lectures se font uniquement par conversation (pattern d'accès simple et prévisible, pas de jointures transverses nécessaires) ; la donnée (liste de messages d'une conversation) se modélise naturellement en document ou en ligne large sans normalisation complexe. Une base relationnelle resterait un choix valable si le produit avait aussi besoin de requêtes transverses complexes (recherche full-text avancée, rapports croisant plusieurs entités) ou de garanties transactionnelles fortes entre plusieurs types de données liées — ce qui n'est pas le cas décrit ici.

## Exercice 5

Le tracing distribué propage un identifiant de trace unique à travers tous les services traversés par une même requête, et enregistre pour chaque "span" (l'appel à un service ou une opération donnée) son horodatage de début/fin. En reconstituant la trace complète d'une requête lente, on visualise directement quel span (quel service, ou quel appel réseau entre deux services) a pris un temps anormalement long — par exemple, un appel vers le service B qui prenait 20 ms et en prend maintenant 400 ms. Des logs isolés par service ne permettent pas cette reconstitution : chaque service ne voit que sa propre exécution, sans identifiant commun reliant les logs entre eux ni vision du temps passé en attente réseau entre les services.
