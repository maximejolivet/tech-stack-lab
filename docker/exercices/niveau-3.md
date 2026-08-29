# Exercices Docker — Niveau 3 (Avancé)

## Exercice 1 — Utilisateur non-root

Modifie le Dockerfile multi-stage de l'exercice 1 (niveau 2) pour que le processus final s'exécute sous un utilisateur non-root plutôt que root par défaut.

## Exercice 2 — Cache mount BuildKit

Explique ce qu'apporte `RUN --mount=type=cache,target=/root/.npm npm ci` par rapport à un simple `RUN npm ci`, et pourquoi ce cache n'alourdit pas l'image finale.

## Exercice 3 — Image minimale

Compare en 2-3 phrases une image basée sur `node:20` (Debian complet), `node:20-alpine`, et une image `distroless` — en termes de taille, de surface d'attaque, et de facilité de débogage.

## Exercice 4 — Filesystem en lecture seule

Modifie la commande `docker run` de l'API pour que son filesystem racine soit en lecture seule (`--read-only`), en explicitant le ou les volumes nécessaires si l'application doit écrire des fichiers temporaires.

## Exercice 5 — Registre et CI/CD

Décris les étapes (en commandes ou en pseudo-YAML) d'un pipeline CI qui build l'image `mon-api`, la tague avec le hash du commit, la pousse vers un registre (ex. GitHub Container Registry), puis explique pourquoi taguer avec le hash du commit plutôt qu'avec `latest` est une bonne pratique.
