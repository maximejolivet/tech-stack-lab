# Exercices Docker — Niveau 1 (Bases)

## Exercice 1 — Premier conteneur

Lance un conteneur `nginx` nommé `mon-nginx`, en arrière-plan, avec le port 8080 de l'hôte mappé vers le port 80 du conteneur. Vérifie qu'il tourne, puis affiche ses logs.

## Exercice 2 — Dockerfile minimal

Écris un `Dockerfile` pour une application Node.js : image de base `node:20-alpine`, dossier de travail `/app`, copie et installation des dépendances avant le reste du code, exposition du port 3000, démarrage via `node server.js`.

## Exercice 3 — Build et run

Construis l'image du Dockerfile de l'exercice 2 avec le tag `mon-api:1.0`, puis lance un conteneur à partir de cette image en mappant le port 3000.

## Exercice 4 — Volume

Crée un volume nommé `pgdata`, puis lance un conteneur `postgres:16` qui utilise ce volume pour stocker ses données à l'emplacement `/var/lib/postgresql/data`.

## Exercice 5 — docker-compose minimal

Écris un `docker-compose.yml` avec deux services : `api` (buildée depuis le Dockerfile local, port 3000) et `db` (image `postgres:16`, variable d'environnement `POSTGRES_PASSWORD`).
