# Docker

## 1. Introduction

Docker est une plateforme de conteneurisation : elle empaquette une application avec toutes ses dépendances (runtime, bibliothèques, configuration) dans une unité isolée et portable, le **conteneur**. Ce dossier suppose les bases Linux acquises (voir [`../linux/`](../linux/)), notamment processus et système de fichiers, puisque Docker s'appuie directement dessus.

**À quoi sert-il ?**
- Garantir qu'une application s'exécute de façon identique en local, en CI, et en production ("ça marche sur ma machine" devient un non-problème).
- Isoler les dépendances de plusieurs projets sur une même machine sans conflit de versions.
- Standardiser le déploiement : une image Docker se déploie de la même façon quel que soit l'hébergeur.

**Où se situe-t-il dans une architecture web ?** Une couche d'exécution au-dessus de Linux (voir [`../linux/`](../linux/)) : chaque conteneur partage le noyau de la machine hôte mais reste isolé (processus, réseau, filesystem) grâce aux namespaces/cgroups. Base de l'orchestration à plus grande échelle (voir [`../kubernetes/`](../kubernetes/)).

**Avantages** : environnements reproductibles, démarrage rapide (contrairement à une VM complète), écosystème d'images prêtes à l'emploi (Docker Hub), standard quasi-universel du déploiement moderne.
**Limites** : overhead de compréhension (réseau, volumes, layers) pour les nouveaux venus, isolation moins forte qu'une VM (partage du noyau hôte), images mal construites peuvent devenir lourdes et lentes à builder/déployer.

## 2. Prérequis

- Bases Linux : système de fichiers, processus, permissions (voir [`../linux/`](../linux/)).
- Ligne de commande à l'aise (navigation, redirections).
- Docker Desktop (ou Docker Engine) installé (`docker --version`).

## 3. Rappel des bases 🟢

### 01 - Images vs conteneurs

**Explication** — Une **image** est un modèle en lecture seule (comme une classe) construit en couches ; un **conteneur** est une instance en cours d'exécution de cette image (comme un objet). Une même image peut donner naissance à plusieurs conteneurs indépendants.

```bash
docker images         # lister les images téléchargées/construites localement
docker ps              # lister les conteneurs en cours d'exécution
docker ps -a             # lister aussi les conteneurs arrêtés
```

### 02 - Lancer un conteneur

**Explication** — `docker run` télécharge l'image si besoin, crée et démarre un conteneur à partir d'elle.

```bash
docker run -d -p 8080:80 --name mon-site nginx   # -d: détaché, -p: mapping de port
docker logs -f mon-site                             # suivre les logs du conteneur
docker exec -it mon-site sh                           # ouvrir un shell dans le conteneur en cours
docker stop mon-site && docker rm mon-site              # arrêter puis supprimer le conteneur
```

**Bonne pratique** : nommer explicitement les conteneurs (`--name`) plutôt que de laisser Docker générer un nom aléatoire — bien plus lisible pour les manipuler ensuite.

### 03 - Écrire un Dockerfile

**Explication** — Un `Dockerfile` décrit, instruction par instruction, comment construire une image. Chaque instruction crée une nouvelle **couche** (layer), mise en cache si son contenu n'a pas changé.

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Erreur fréquente** : faire `COPY . .` avant `RUN npm ci` — chaque modification du code source invaliderait alors le cache de l'installation des dépendances, qui serait réinstallée à chaque build. Toujours copier les fichiers de dépendances (`package*.json`) et installer **avant** de copier le reste du code.

### 04 - Builder une image

**Explication** — `docker build` exécute les instructions du Dockerfile pour produire une image taguée.

```bash
docker build -t mon-app:1.0 .
docker run -d -p 3000:3000 mon-app:1.0
```

**Bonne pratique** : utiliser un fichier `.dockerignore` (même syntaxe que `.gitignore`) pour exclure `node_modules/`, `.git/`, fichiers `.env` — évite d'alourdir le contexte de build et de fuiter des secrets dans l'image.

### 05 - Volumes (persistance des données)

**Explication** — Le système de fichiers d'un conteneur est éphémère par défaut : toute donnée écrite disparaît à sa suppression. Un **volume** monte un espace de stockage persistant, géré par Docker, indépendant du cycle de vie du conteneur.

```bash
docker volume create pgdata
docker run -d -v pgdata:/var/lib/postgresql/data postgres:16
```

**Erreur fréquente** : stocker les données d'une base de données directement dans le filesystem du conteneur sans volume — un `docker rm` accidentel efface alors définitivement toutes les données.

### 06 - Réseau de base

**Explication** — Par défaut, Docker crée un réseau bridge isolé : les conteneurs peuvent communiquer entre eux via leur nom si placés sur le même réseau, mais ne sont accessibles depuis l'extérieur que via un mapping de port explicite (`-p hôte:conteneur`).

```bash
docker network create mon-reseau
docker run -d --network mon-reseau --name db postgres:16
docker run -d --network mon-reseau --name api mon-api   # "api" peut joindre "db" par son nom
```

### 07 - Docker Compose (introduction)

**Explication** — `docker-compose.yml` décrit un ensemble de conteneurs liés (application + base de données + cache) et leur configuration, démarrables/arrêtables ensemble en une commande.

```yaml
# docker-compose.yml
services:
  api:
    build: .
    ports: ["3000:3000"]
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes: ["pgdata:/var/lib/postgresql/data"]
volumes:
  pgdata:
```

```bash
docker compose up -d      # démarrer tous les services en arrière-plan
docker compose down         # arrêter et supprimer les conteneurs (les volumes nommés persistent)
```

**Bonne pratique** : `depends_on` garantit l'ordre de démarrage des conteneurs, mais pas que le service dépendant est réellement *prêt* (ex. Postgres qui accepte encore des connexions) — pour ça, coupler avec un `healthcheck` (voir section avancée).

## 4. Concepts intermédiaires 🟡

- **Multi-stage builds** : utiliser plusieurs étapes `FROM` dans un même Dockerfile pour séparer l'environnement de build (lourd : compilateurs, dépendances de dev) de l'image finale (légère, seulement le résultat).

```dockerfile
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**Bonne pratique** : toujours viser l'image finale la plus petite possible (base `alpine` ou `distroless` quand c'est compatible) — réduit la surface d'attaque et accélère les déploiements.

- **Cache des couches (layer caching)** : Docker réutilise une couche mise en cache tant que l'instruction et son contexte n'ont pas changé — ordonner le Dockerfile du moins volatile (dépendances) au plus volatile (code source) maximise les hits de cache en CI.
- **Docker Compose multi-service** : orchestrer plusieurs services applicatifs (API, worker, base de données, cache Redis) avec des réseaux et volumes partagés, souvent avec des fichiers `docker-compose.override.yml` distincts par environnement.
- **Healthchecks** : déclarer une commande qui vérifie qu'un conteneur est réellement opérationnel, pas seulement démarré.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1
```

- **Limites de ressources** : contraindre la mémoire et le CPU d'un conteneur pour éviter qu'il n'affame les autres sur la même machine (`docker run --memory=512m --cpus=1`).
- **Sécurité de base** : ne jamais exécuter un processus en `root` dans le conteneur (instruction `USER` dans le Dockerfile), scanner les images pour des vulnérabilités connues (`docker scout` ou Trivy), ne jamais mettre de secrets en dur dans une image (ils resteraient dans l'historique des couches même supprimés ensuite).
- **Modes réseau** : `bridge` (par défaut, isolé avec NAT), `host` (le conteneur partage directement la pile réseau de l'hôte, pas de mapping de port nécessaire mais isolation réduite), `none` (aucun accès réseau).

## 5. Concepts avancés 🟠🔴

- **Registre d'images et CI/CD** : `docker push`/`docker pull` vers un registre (Docker Hub, GitHub Container Registry, registre privé) — pilier de tout pipeline CI/CD qui build une image et la déploie (voir [`../ci-cd/`](../ci-cd/)).
- **Optimisation avancée du cache de build** : BuildKit (moteur de build par défaut moderne) permet le cache mount (`--mount=type=cache`) pour persister un cache de dépendances (ex. cache npm/pip) entre builds sans l'inclure dans l'image finale.
- **Distroless et images minimales** : images sans shell ni gestionnaire de paquets, réduisant drastiquement la surface d'attaque — compromis à évaluer avec le besoin de débogage en production.
- **Orchestration native (Docker Swarm)** : mode cluster intégré à Docker pour répartir des conteneurs sur plusieurs machines — largement supplanté par Kubernetes en production (voir [`../kubernetes/`](../kubernetes/)) mais plus simple pour de petites infrastructures.
- **Observabilité** : agréger les logs de conteneurs (driver de logging vers un collecteur centralisé plutôt que `docker logs` par conteneur), exposer des métriques (`docker stats`, ou exporter vers Prometheus).
- **Sécurité approfondie** : signature d'images (Docker Content Trust), politiques d'admission qui refusent les images non scannées, exécution en lecture seule du filesystem racine du conteneur (`--read-only` + volumes explicites pour ce qui doit rester inscriptible) — voir [`../security/`](../security/).

## 6. Commandes / syntaxe à connaître

```bash
docker build -t nom:tag .              # construire une image
docker run -d -p host:conteneur nom     # lancer un conteneur
docker ps -a                             # lister tous les conteneurs
docker logs -f nom                        # suivre les logs
docker exec -it nom sh                     # shell dans un conteneur
docker stop nom && docker rm nom            # arrêter et supprimer
docker volume ls / docker network ls         # lister volumes / réseaux
docker compose up -d / down                   # démarrer / arrêter une stack
docker system prune -a                          # nettoyer images/conteneurs inutilisés
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Conteneurisation d'une petite application API + base de données**

- Un `Dockerfile` multi-stage pour une API Node.js (ou équivalente) qui build puis produit une image finale `alpine` légère, exécutée sous un utilisateur non-root.
- Un `docker-compose.yml` avec deux services : `api` (buildée depuis le Dockerfile) et `db` (Postgres officiel), reliés par un réseau Compose, avec un volume nommé pour la persistance des données.
- Un `HEALTHCHECK` sur le service `api` qui vérifie une route `/health`.
- Un `.dockerignore` excluant `node_modules/`, `.git/` et les fichiers `.env`.

Bonus : ajouter un service `nginx` en reverse proxy devant l'API, limiter la mémoire/CPU de chaque service dans le `docker-compose.yml`, et scanner l'image finale avec Trivy ou `docker scout`.

## Checklist

- [ ] Comprendre les fondamentaux (image vs conteneur, Dockerfile, layers)
- [ ] Savoir construire une image et lancer un conteneur avec les bons ports/volumes
- [ ] Maîtriser la syntaxe principale (`docker run`, `docker build`, `docker compose`)
- [ ] Comprendre les concepts importants (multi-stage builds, cache de couches, healthchecks)
- [ ] Savoir debugger (`docker logs`, `docker exec`, `docker inspect`)
- [ ] Connaître les bonnes pratiques (utilisateur non-root, `.dockerignore`, images minimales)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (BuildKit cache mount, distroless, sécurité des images)

## 10. Ressources

- [Documentation officielle Docker](https://docs.docker.com/) — référence complète.
- [Dockerfile best practices](https://docs.docker.com/build/building/best-practices/) — guide officiel d'optimisation.
- [Docker Compose — référence](https://docs.docker.com/compose/) — syntaxe complète du fichier Compose.
- [roadmap.sh — Docker](https://roadmap.sh/docker) — vue d'ensemble structurée des compétences à couvrir.
