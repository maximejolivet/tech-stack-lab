# Node.js — Solutions niveau 3

## 3.1 — Serveur avec `cluster`

```js
import cluster from 'node:cluster';
import os from 'node:os';
import http from 'node:http';

if (cluster.isPrimary) {
  const nbCoeurs = os.cpus().length;
  for (let i = 0; i < nbCoeurs; i++) cluster.fork();

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} arrêté, redémarrage`);
    cluster.fork();
  });
} else {
  http.createServer((req, res) => {
    res.end(`Répondu par le process ${process.pid}`);
  }).listen(3000);
}
```

Chaque requête HTTP est distribuée par le process primaire aux workers (round-robin sur la plupart des OS) : rafraîchir plusieurs fois montre des PID différents.

## 3.2 — Détecter une fuite mémoire

```js
// fuite.js — volontairement fautif
const cache = []; // jamais vidé : fuite

function traiterRequete() {
  cache.push(new Array(100_000).fill('x')); // accumulation sans limite
}

setInterval(traiterRequete, 100);
```

Lancé avec `node --inspect fuite.js`, l'onglet Memory de Chrome DevTools (heap snapshots pris toutes les X secondes) montre une taille de heap qui croît indéfiniment.

**Correction** — borner le cache (LRU, TTL, ou taille max) :

```js
const cache = [];
const TAILLE_MAX = 100;

function traiterRequete() {
  if (cache.length >= TAILLE_MAX) cache.shift(); // retire le plus ancien
  cache.push(new Array(100_000).fill('x'));
}
```

## 3.3 — Graceful shutdown

```js
import http from 'node:http';

const server = http.createServer((req, res) => {
  setTimeout(() => res.end('Réponse lente'), 3000);
});

server.listen(3000);

process.on('SIGTERM', () => {
  console.log('SIGTERM reçu, arrêt en cours...');
  server.close(() => {
    console.log('Toutes les requêtes en cours terminées, process quitte');
    process.exit(0);
  });
  // server.close() n'accepte plus de nouvelles connexions,
  // mais attend que les requêtes déjà en cours se terminent avant le callback.
});
```

Test : lancer une requête lente, puis `kill -TERM <pid>` (ou `Ctrl+C` si le handler est branché sur `SIGINT`) pendant qu'elle est en cours — le serveur répond avant de quitter.

## 3.4 — Worker thread pour un calcul lourd

```js
// calcul-lourd.js
import { parentPort, workerData } from 'node:worker_threads';

function calculerLourd(n) {
  let total = 0;
  for (let i = 0; i < n; i++) total += i;
  return total;
}

parentPort.postMessage(calculerLourd(workerData.n));
```

```js
// serveur.js
import http from 'node:http';
import { Worker } from 'node:worker_threads';

http.createServer((req, res) => {
  if (req.url === '/lourd') {
    const worker = new Worker('./calcul-lourd.js', { workerData: { n: 5_000_000_000 } });
    worker.on('message', (result) => res.end(`Résultat : ${result}`));
    return;
  }
  res.end('OK, toujours réactif');
}).listen(3000);
```

Sans `worker_threads`, exécuter `calculerLourd` directement dans le handler bloque l'event loop : `/` (l'autre route) ne répond plus tant que le calcul tourne. Avec le worker, `/` reste réactif car le calcul tourne dans un thread séparé.

## 3.5 — Rate limiter maison

```js
import http from 'node:http';

const FENETRE_MS = 10_000;
const MAX_REQUETES = 5;
const compteurs = new Map(); // ip -> { count, resetAt }

function rateLimiter(ip) {
  const maintenant = Date.now();
  const entree = compteurs.get(ip);

  if (!entree || maintenant > entree.resetAt) {
    compteurs.set(ip, { count: 1, resetAt: maintenant + FENETRE_MS });
    return true;
  }
  if (entree.count >= MAX_REQUETES) return false;
  entree.count++;
  return true;
}

http.createServer((req, res) => {
  const ip = req.socket.remoteAddress;
  if (!rateLimiter(ip)) {
    res.writeHead(429, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Too Many Requests' }));
    return;
  }
  res.end('OK');
}).listen(3000);
```

En production, un `Map` en mémoire ne suffit pas au-delà d'une seule instance (chaque process aurait son propre compteur) — on utiliserait Redis (voir `../redis/`) comme store partagé.
