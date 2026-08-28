# Node.js — Solutions niveau 1

## 1.1 — Lire un fichier

```js
import fs from 'node:fs/promises';

async function lire() {
  try {
    const contenu = await fs.readFile('data.txt', 'utf-8');
    console.log(contenu);
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.error('Le fichier data.txt est introuvable.');
    } else {
      console.error('Erreur de lecture :', err.message);
    }
  }
}

lire();
```

## 1.2 — Serveur HTTP minimal

```js
import http from 'node:http';

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello Node');
    return;
  }
  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(3000, () => console.log('http://localhost:3000'));
```

## 1.3 — Arguments de la ligne de commande

```js
// salut.js
const nom = process.argv[2]; // 0 = node, 1 = salut.js, 2 = premier argument
console.log(`Bonjour ${nom} !`);
```

```bash
node salut.js Max
# Bonjour Max !
```

## 1.4 — CommonJS vers ESM

```js
import path from 'node:path';

export function joinPaths(a, b) {
  return path.join(a, b);
}
```

Dans `package.json`, il faut ajouter `"type": "module"` pour que Node interprète les fichiers `.js` comme de l'ESM (sinon `import`/`export` lèvent une erreur de syntaxe). Alternative : renommer le fichier en `.mjs`.

## 1.5 — Script npm

```json
{
  "scripts": {
    "dev": "node --watch app.js",
    "start": "node app.js"
  }
}
```

```bash
npm run dev     # mode développement avec rechargement auto
npm start       # "start" est un script npm spécial, pas besoin de "run"
```
