# Node.js — Solutions niveau 2

## 2.1 — EventEmitter personnalisé

```js
import { EventEmitter } from 'node:events';

class Panier extends EventEmitter {
  constructor() {
    super();
    this.produits = [];
  }
  ajouter(produit) {
    this.produits.push(produit);
    this.emit('produit:ajoute', produit);
  }
}

const panier = new Panier();
panier.on('produit:ajoute', (produit) => console.log(`Ajouté : ${produit}`));
panier.ajouter('Clavier');
```

## 2.2 — Petit serveur avec routing JSON

```js
import http from 'node:http';

const users = [{ id: 1, nom: 'Max' }];

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(users));
    return;
  }

  if (req.method === 'POST' && req.url === '/users') {
    let body = '';
    req.on('data', (chunk) => { body += chunk; });
    req.on('end', () => {
      const data = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify(data));
    });
    return;
  }

  res.writeHead(404);
  res.end();
});

server.listen(3000);
```

## 2.3 — Compter les lignes avec un stream

```js
import fs from 'node:fs';
import readline from 'node:readline';

async function compterLignes(fichier) {
  const rl = readline.createInterface({ input: fs.createReadStream(fichier) });
  let count = 0;
  for await (const _ligne of rl) count++;
  return count;
}

compterLignes('gros-fichier.log').then((n) => console.log(`${n} lignes`));
```

`readline` s'appuie sur un stream : le fichier est lu morceau par morceau, jamais chargé entièrement en mémoire.

## 2.4 — Gestion d'erreur async

```js
async function recupererDonnees() {
  return new Promise((_resolve, reject) => {
    setTimeout(() => reject(new Error('Timeout')), 100);
  });
}

async function main() {
  try {
    await recupererDonnees();
  } catch (err) {
    console.error('Erreur gérée localement :', err.message);
  }
}
main();

// Filet de sécurité si un appel oublie le try/catch ailleurs dans l'app
process.on('unhandledRejection', (err) => {
  console.error('Rejet non géré :', err);
});
```

## 2.5 — Configuration via `.env`

```bash
# .env
PORT=4000
```

```js
import 'dotenv/config';
import http from 'node:http';

const port = process.env.PORT ?? 3000;
http.createServer((req, res) => res.end('OK')).listen(port, () => {
  console.log(`Serveur sur le port ${port}`);
});
```

```gitignore
.env
```
