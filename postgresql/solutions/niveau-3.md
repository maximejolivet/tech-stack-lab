# Solutions — Niveau 3 (Avancé)

## Exercice 1

Sous MVCC, chaque `UPDATE` ne modifie pas la ligne en place : il crée une **nouvelle version** de la ligne et marque l'ancienne comme obsolète (mais elle reste physiquement sur le disque tant qu'aucune transaction en cours n'en a potentiellement besoin). Avec des milliers d'`UPDATE`/minute, ces versions obsolètes s'accumulent plus vite qu'elles ne sont nettoyées, gonflant la taille physique de la table ("bloat"). `autovacuum` est le processus qui parcourt périodiquement les tables et marque l'espace occupé par ces versions obsolètes comme réutilisable pour de futures écritures — sans lui (ou mal configuré/trop lent pour le rythme d'écriture), le bloat croît indéfiniment.

## Exercice 2

`EXPLAIN` seul affiche le plan d'exécution **estimé** par l'optimiseur, sans exécuter réellement la requête. `EXPLAIN ANALYZE` exécute réellement la requête et ajoute les temps effectifs et le nombre de lignes réel à chaque étape du plan, permettant de repérer les écarts entre estimation et réalité (signe de statistiques obsolètes). Précaution sur un `DELETE`/`UPDATE` : `EXPLAIN ANALYZE` exécute réellement la requête, donc les lignes sont effectivement supprimées/modifiées — pour analyser un tel plan sans effet de bord, on l'englobe dans une transaction qu'on annule ensuite (`BEGIN; EXPLAIN ANALYZE DELETE ...; ROLLBACK;`).

## Exercice 3

Sous `REPEATABLE READ`, chaque transaction voit un instantané cohérent, mais deux transactions concurrentes peuvent chacune lire un solde total de 9 000€, ajouter chacune 2 000€ (chacune validant la règle "< 10 000€" sur la base de sa propre lecture), et committer toutes les deux — résultat final 13 000€, violant la règle métier alors qu'aucune des deux transactions n'a "vu" l'autre. `SERIALIZABLE` détecte ce type de conflit de sérialisation (le résultat final ne correspond à aucun ordre séquentiel possible des deux transactions) et force l'une des deux à échouer au `COMMIT`, à charge pour l'application de la retenter — garantissant que le résultat final est équivalent à une exécution strictement séquentielle.

## Exercice 4

Utiliser une colonne de type `GEOGRAPHY` (ou `GEOMETRY`) fournie par l'extension `PostGIS` (`CREATE EXTENSION postgis;`), stockant les coordonnées de chaque magasin. La requête de proximité s'appuie typiquement sur `ST_DWithin(location, ST_MakePoint(lon, lat)::geography, 5000)` (rayon en mètres), idéalement accéléré par un index spatial `GIST` sur la colonne `location`.

## Exercice 5

La réplication logique réplique les changements au niveau **ligne** (via un flux de type `INSERT`/`UPDATE`/`DELETE` logiques), ce qui permet de choisir précisément quelle(s) table(s) répliquer (`CREATE PUBLICATION` ciblée sur `products` uniquement) et fonctionne entre versions PostgreSQL différentes, contrairement à la réplication physique qui copie l'état binaire des fichiers de toute l'instance et exige des versions strictement compatibles des deux côtés.
