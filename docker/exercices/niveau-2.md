# Exercices Docker — Niveau 2 (Intermédiaire)

## Exercice 1 — Multi-stage build

Transforme le Dockerfile de l'exercice 2 (niveau 1) en build multi-stage : une étape `build` qui installe toutes les dépendances et build l'application, puis une étape finale légère qui ne copie que le résultat du build.

## Exercice 2 — .dockerignore

Écris un fichier `.dockerignore` qui exclut `node_modules`, `.git`, et tout fichier `.env` du contexte de build.

## Exercice 3 — Healthcheck

Ajoute une instruction `HEALTHCHECK` au Dockerfile de l'API qui vérifie toutes les 30 secondes que la route `http://localhost:3000/health` répond, avec un timeout de 3 secondes.

## Exercice 4 — Réseau entre conteneurs

Crée un réseau Docker nommé `backend`, puis lance deux conteneurs (`db` et `api`) sur ce réseau. Explique comment le conteneur `api` peut joindre `db` sans connaître son adresse IP.

## Exercice 5 — Limites de ressources

Modifie le service `api` du `docker-compose.yml` de l'exercice 5 (niveau 1) pour limiter sa mémoire à 512 Mo et son CPU à 1.
