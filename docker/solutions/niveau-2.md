# Solutions — Niveau 2 (Intermédiaire)

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
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

## Exercice 2

```
node_modules
.git
**/.env
```

## Exercice 3

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1
```

## Exercice 4

```bash
docker network create backend
docker run -d --network backend --name db postgres:16
docker run -d --network backend --name api mon-api:1.0
```

Sur un réseau Docker créé par l'utilisateur, Docker fournit une résolution DNS interne automatique : le conteneur `api` peut simplement joindre `db` en utilisant son nom (`db:5432`) comme s'il s'agissait d'un nom d'hôte, sans connaître son adresse IP interne (qui peut d'ailleurs changer à chaque redémarrage).

## Exercice 5

```yaml
services:
  api:
    build: .
    ports: ["3000:3000"]
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1"
```
