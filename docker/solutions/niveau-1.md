# Solutions — Niveau 1 (Bases)

## Exercice 1

```bash
docker run -d -p 8080:80 --name mon-nginx nginx
docker ps
docker logs mon-nginx
```

## Exercice 2

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

## Exercice 3

```bash
docker build -t mon-api:1.0 .
docker run -d -p 3000:3000 mon-api:1.0
```

## Exercice 4

```bash
docker volume create pgdata
docker run -d -v pgdata:/var/lib/postgresql/data postgres:16
```

## Exercice 5

```yaml
services:
  api:
    build: .
    ports: ["3000:3000"]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
```
