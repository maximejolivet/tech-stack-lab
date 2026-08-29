# Solutions — Niveau 3 (Avancé)

## Exercice 1

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

## Exercice 2

`--mount=type=cache,target=/root/.npm` déclare un cache BuildKit persistant entre les builds pour le dossier de cache npm : les paquets déjà téléchargés lors d'un build précédent sont réutilisés, accélérant les installations suivantes même quand `package.json` a changé (contrairement au cache de couche classique qui invalide toute la couche `RUN` dès que son contexte change). Ce cache est stocké séparément par BuildKit et n'est **jamais** inclus dans l'image finale — seul le résultat de `npm ci` (le dossier `node_modules`) l'est.

## Exercice 3

`node:20` (basée sur Debian) est la plus volumineuse (plusieurs centaines de Mo) mais offre un shell complet et un gestionnaire de paquets, pratique pour déboguer. `node:20-alpine` est nettement plus légère (musl libc, BusyBox) avec un shell minimal, bon compromis taille/débogabilité pour la plupart des projets. Une image `distroless` ne contient ni shell ni gestionnaire de paquets — la plus petite et la plus sûre (surface d'attaque minimale), mais impossible d'y faire un `docker exec ... sh` pour investiguer un problème en direct.

## Exercice 4

```bash
docker run -d --read-only --tmpfs /tmp -p 3000:3000 mon-api:1.0
```

`--tmpfs /tmp` fournit un espace en mémoire inscriptible pour les fichiers temporaires de l'application, tandis que le reste du filesystem du conteneur reste strictement en lecture seule.

## Exercice 5

```yaml
# pipeline CI (pseudo-syntaxe)
- name: Build image
  run: docker build -t ghcr.io/mon-org/mon-api:${{ github.sha }} .
- name: Push image
  run: |
    echo "$REGISTRY_TOKEN" | docker login ghcr.io -u user --password-stdin
    docker push ghcr.io/mon-org/mon-api:${{ github.sha }}
```

Taguer avec le hash du commit (plutôt que `latest`) rend chaque déploiement traçable et reproductible : on sait exactement quel code tourne en production, on peut redéployer une version précédente précise en cas de rollback, et on évite le risque de `latest` qui pointe silencieusement vers une image différente selon le moment où elle a été pull.
