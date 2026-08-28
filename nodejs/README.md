# Node.js

## 1. Introduction

Node.js est un **runtime JavaScript côté serveur**, construit sur le moteur V8 de Chrome. Il permet d'exécuter du JS en dehors du navigateur : scripts, serveurs HTTP, CLI, outils de build. Ce dossier suppose le langage JS déjà acquis (voir [`../javascript/`](../javascript/)) et se concentre sur ce que Node ajoute : un environnement d'exécution, des APIs système, un modèle I/O non-bloquant.

**À quoi sert-il ?**
- Écrire des API/serveurs web en JS (même langage que le front).
- Scripts d'automatisation, outils CLI, tooling de build (bundlers, linters tournent sous Node).
- I/O intensif à haute concurrence (beaucoup de requêtes qui attendent réseau/disque plutôt que de calculer).

**Où se situe-t-il dans une architecture web ?** Couche backend : serveur HTTP, accès base de données, logique métier, exposition d'API consommées par le front (JS/mobile) ou d'autres services.

**Avantages** : un seul langage front/back, écosystème npm immense, excellent pour l'I/O concurrent (event loop non-bloquant), démarrage rapide, très adapté aux microservices légers.

**Limites** : mono-thread pour le JS applicatif → le calcul CPU-intensif bloque tout (parade : `worker_threads`, ou déléguer à un autre service) ; typage dynamique (mêmes pièges que JS, mitigés par TypeScript) ; callback hell historique si on n'utilise pas `async`/`await`.

> Ce dossier ne couvre pas un framework backend précis (Express, Nest...) — il pose les fondations Node communes à tous. Les exemples utilisent le module `http` natif pour rester indépendants d'un framework.

## 2. Prérequis

- JavaScript solide : scope, fonctions, closures, Promises, `async`/`await` — voir [`../javascript/`](../javascript/).
- Node.js installé ([nodejs.org](https://nodejs.org), version LTS recommandée) et un terminal.
- Notions HTTP de base (méthodes, status codes, headers) utiles pour la partie serveur.

## 3. Rappel des bases 🟢

### 01 - Qu'est-ce que Node.js

**Explication** — Node embarque le moteur V8 (compile le JS en code machine) et l'entoure d'une **libuv** qui gère l'I/O asynchrone (fichiers, réseau, timers) via une boucle d'événements (event loop) et un pool de threads système en coulisses. Le code JS applicatif reste single-threaded : c'est l'I/O qui est déléguée en arrière-plan, pas le JS lui-même.

**Cas d'usage** : tout ce qui attend beaucoup (requêtes réseau, lecture disque, appels DB) plutôt que de calculer beaucoup.

**Erreur fréquente** : croire que Node est multi-thread par défaut pour le code applicatif — un calcul synchrone lourd (boucle de tri géante, hashing) bloque l'event loop et donc **toutes** les requêtes en cours.

**Bonne pratique** : garder le code applicatif non-bloquant ; déporter le calcul lourd (`worker_threads`, service externe, queue).

### 02 - Node vs JS navigateur

**Explication** — Même langage, environnement différent : pas de `window`, `document`, DOM. Node expose ses propres globals.

```js
// CommonJS (par défaut, extension .js sans "type": "module")
console.log(__dirname);   // chemin absolu du dossier du fichier
console.log(__filename);  // chemin absolu du fichier
console.log(process.argv); // arguments de la ligne de commande
console.log(process.env.NODE_ENV); // variables d'environnement

// ESM (avec "type": "module" dans package.json, ou extension .mjs)
console.log(import.meta.url); // équivalent __filename en ESM
```

**Erreur fréquente** : utiliser `__dirname` dans un fichier `.mjs`/ESM — n'existe pas, il faut le reconstruire via `import.meta.url` + `fileURLToPath`.

### 03 - Modules : CommonJS vs ESM

```js
// CommonJS (require/module.exports) — historique, encore très répandu
const fs = require('fs');
module.exports = { maFonction };

// ESM (import/export) — standard moderne, aligné sur le JS navigateur
import fs from 'node:fs';
export function maFonction() {}
```

**Cas d'usage** : ESM par défaut sur un nouveau projet (`"type": "module"` dans `package.json`) pour rester cohérent avec le JS moderne et le front ; CommonJS reste courant dans du code legacy ou certains packages npm anciens.

**Erreur fréquente** : mélanger les deux dans le même fichier — un fichier est soit CommonJS, soit ESM, pas les deux (l'interop existe mais a des subtilités, ex. `import` d'un module CJS fonctionne, l'inverse est plus limité).

**Bonne pratique** : préfixer les imports de modules core avec `node:` (`import fs from 'node:fs'`) — plus explicite, évite toute ambiguïté avec un package npm du même nom.

### 04 - npm & package.json

**Explication** — `package.json` décrit le projet : dépendances, scripts, métadonnées. `package-lock.json` fige les versions exactes résolues (à committer, garantit des installs reproductibles).

```json
{
  "name": "mon-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "node src/index.js",
    "test": "node --test"
  },
  "dependencies": { "express": "^4.19.0" },
  "devDependencies": { "nodemon": "^3.0.0" }
}
```

**`dependencies` vs `devDependencies`** : `dependencies` = nécessaire en production (ex. un framework HTTP) ; `devDependencies` = uniquement pour développer (linter, testeur, watcher).

**Semver (`^`, `~`)** : `^1.2.3` accepte les mises à jour mineures/patch (`<2.0.0`), `~1.2.3` accepte seulement les patchs (`<1.3.0`).

**Erreur fréquente** : ne pas committer `package-lock.json` → versions de dépendances qui divergent entre les machines de l'équipe.

**Bonne pratique** : `npm ci` (au lieu de `npm install`) en CI/déploiement — installe strictement depuis le lockfile, plus rapide et reproductible.

### 05 - Installer et gérer des dépendances

```bash
npm install express          # ajoute à dependencies
npm install -D nodemon       # ajoute à devDependencies (-D = --save-dev)
npm uninstall express
npm outdated                 # dépendances à mettre à jour
npm audit                    # vulnérabilités connues
```

**Erreur fréquente** : installer des packages globalement (`-g`) pour des besoins de projet — casse la reproductibilité (chaque machine peut avoir une version différente). Réserver `-g` aux vrais outils CLI système.

### 06 - Modules core essentiels

```js
import fs from 'node:fs';       // système de fichiers
import path from 'node:path';   // manipulation de chemins
import os from 'node:os';       // infos système (CPU, mémoire, OS)
import http from 'node:http';   // serveur/client HTTP bas niveau
import { EventEmitter } from 'node:events'; // pattern pub/sub
```

**Bonne pratique** : `path.join()` / `path.resolve()` plutôt que la concaténation de chaînes pour construire des chemins — gère les séparateurs `/` vs `\` selon l'OS.

```js
path.join('dossier', 'fichier.txt');       // 'dossier/fichier.txt' (ou '\' sur Windows)
path.resolve('dossier', 'fichier.txt');    // chemin absolu depuis le cwd
```

### 07 - Lire et écrire des fichiers

```js
import fs from 'node:fs/promises';

// asynchrone (à privilégier : ne bloque pas l'event loop)
const contenu = await fs.readFile('data.json', 'utf-8');
await fs.writeFile('sortie.txt', 'contenu');

// synchrone (bloque tout le process — réservé aux scripts CLI simples, jamais dans un serveur)
import { readFileSync } from 'node:fs';
const c = readFileSync('data.json', 'utf-8');
```

**Erreur fréquente** : utiliser les versions `*Sync` dans un serveur HTTP — bloque l'event loop, donc toutes les requêtes en cours, pendant chaque I/O disque.

**Bonne pratique** : `fs/promises` + `async`/`await` par défaut ; réserver les fonctions sync aux scripts one-shot exécutés au démarrage (avant de servir du trafic).

### 08 - Créer un serveur HTTP minimal

```js
import http from 'node:http';

const server = http.createServer((req, res) => {
  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'Hello' }));
    return;
  }
  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(3000, () => console.log('Serveur sur http://localhost:3000'));
```

**Cas d'usage** : comprendre ce mécanisme bas niveau avant d'adopter un framework (Express, Fastify...) qui l'encapsule avec du routing, des middlewares, etc.

### 09 - `process`

```js
process.argv;        // ['node', 'script.js', 'arg1', 'arg2', ...]
process.env.NODE_ENV; // variable d'environnement
process.exit(1);      // quitte avec un code d'erreur
process.cwd();        // dossier d'exécution courant
```

**Bonne pratique** : lire la config sensible (clés API, URL DB) via `process.env`, jamais en dur dans le code — voir aussi les variables d'environnement en section 4.

## 4. Concepts intermédiaires 🟡

**L'event loop en détail** — la boucle tourne en phases successives, répétées en continu :

```text
┌───────────────┐
│    timers     │  setTimeout / setInterval arrivés à échéance
├───────────────┤
│ pending callbacks │ callbacks I/O différés
├───────────────┤
│  poll         │  récupère les nouveaux événements I/O, exécute leurs callbacks
├───────────────┤
│  check        │  setImmediate()
├───────────────┤
│ close callbacks │  ex. socket.on('close', ...)
└───────────────┘
```

Entre **chaque** callback (et entre chaque phase), Node vide entièrement la **microtask queue** : `Promise` callbacks et `process.nextTick` (qui passe même avant les autres microtasks).

```js
console.log('1');
setTimeout(() => console.log('2 (macrotask)'), 0);
Promise.resolve().then(() => console.log('3 (microtask)'));
process.nextTick(() => console.log('4 (nextTick, priorité max)'));
console.log('5');
// Ordre réel : 1, 5, 4, 3, 2
```

**Erreur fréquente** : abuser de `process.nextTick` en récursif → affame l'event loop, les phases timers/poll ne s'exécutent jamais ("I/O starvation").

**Streams & buffers** — traiter des données au fil de l'eau plutôt que tout charger en mémoire :

```js
import fs from 'node:fs';

const lecture = fs.createReadStream('gros-fichier.log');
const ecriture = fs.createWriteStream('sortie.log');
lecture.pipe(ecriture); // gère automatiquement le backpressure
```

**Cas d'usage** : fichiers volumineux, flux réseau, transformation de données en pipeline — évite de charger un fichier de plusieurs Go en RAM.

**Variables d'environnement** — via un fichier `.env` (jamais committé, voir `.gitignore`) chargé au démarrage :

```bash
# .env
DATABASE_URL=postgres://localhost/mabase
PORT=3000
```

```js
import 'dotenv/config'; // package dotenv
const port = process.env.PORT ?? 3000;
```

**Debugging** :

```bash
node --inspect app.js        # ouvre le debugger, à connecter via chrome://inspect
node --inspect-brk app.js    # pause dès la première ligne
node --watch app.js          # relance automatique au changement de fichier (Node 18+)
```

**Gestion d'erreurs asynchrones robuste** :

```js
async function main() {
  try {
    const data = await fetchData();
  } catch (err) {
    console.error('Erreur métier :', err.message);
  }
}

process.on('unhandledRejection', (err) => {
  console.error('Promise rejetée non gérée :', err);
  process.exit(1); // en production : logger puis quitter proprement, ne pas laisser un état incohérent tourner
});
process.on('uncaughtException', (err) => {
  console.error('Exception non attrapée :', err);
  process.exit(1);
});
```

**Erreur fréquente** : oublier `await` sur une fonction async dans un contexte non-catché → `UnhandledPromiseRejection` silencieuse en dev, crash en prod si non géré.

**`EventEmitter`** — base du pattern pub/sub dans Node (streams, `http.Server`, etc. en héritent) :

```js
import { EventEmitter } from 'node:events';

class Commandes extends EventEmitter {}
const bus = new Commandes();

bus.on('commande:creee', (id) => console.log(`Commande ${id} créée`));
bus.emit('commande:creee', 42);
```

**API HTTP avec routing basique** (sans framework, pour comprendre le mécanisme) :

```js
import http from 'node:http';

const routes = {
  'GET /': (req, res) => res.end('Accueil'),
  'GET /users': (req, res) => res.end(JSON.stringify([{ id: 1 }])),
};

const server = http.createServer((req, res) => {
  const handler = routes[`${req.method} ${req.url}`];
  handler ? handler(req, res) : (res.statusCode = 404, res.end('Not found'));
});
```

**Bonne pratique** : en pratique professionnelle, un framework (Express, Fastify) gère le routing, le parsing du body, les middlewares — le réinventer sert à comprendre, pas à produire.

## 5. Concepts avancés 🟠🔴

**Concurrence réelle : `cluster` et `worker_threads`**

- `cluster` : fait tourner plusieurs **process** Node (un par cœur CPU), qui partagent le même port — chaque process a sa propre mémoire, adapté pour scaler un serveur HTTP.
- `worker_threads` : fait tourner du JS dans des **threads** partageant le même process (mémoire partageable via `SharedArrayBuffer`) — adapté pour déporter un calcul CPU-intensif sans bloquer l'event loop principal.

```js
import { Worker } from 'node:worker_threads';

const worker = new Worker('./calcul-lourd.js', { workerData: { n: 1_000_000 } });
worker.on('message', (result) => console.log('Résultat :', result));
```

**Gestion mémoire et fuites courantes** — le garbage collector de V8 libère automatiquement, mais des références oubliées empêchent la libération :

- Écouteurs d'événements jamais retirés (`emitter.on()` sans `.off()` correspondant).
- Caches en mémoire (objets/`Map`) qui grossissent sans jamais être vidés.
- Closures qui capturent involontairement de grosses structures.

**Bonne pratique** : `node --inspect` + onglet Memory de Chrome DevTools pour prendre des heap snapshots et comparer leur évolution dans le temps.

**Performance** — profiler avant d'optimiser (`node --prof`, `clinic.js`), surveiller l'**event loop lag** (délai entre l'attendu et l'exécution réelle d'un `setImmediate`) comme indicateur de saturation.

**Sécurité de base** : `npm audit` régulièrement, ne jamais committer de secrets (`.env` dans `.gitignore`), valider/sanitizer toute entrée utilisateur avant de l'utiliser (évite l'injection), attention à `eval`/`Function()` sur des données externes.

**Architecture Node en production** :
- Process manager (PM2, ou orchestrateur comme Kubernetes — voir `../kubernetes/`) pour le restart automatique et le multi-instance.
- Logs structurés (JSON) plutôt que `console.log` brut, pour être exploitables par un système de collecte.
- **Graceful shutdown** : intercepter `SIGTERM`, cesser d'accepter de nouvelles requêtes, laisser les requêtes en cours se terminer, puis quitter.

```js
process.on('SIGTERM', async () => {
  server.close(() => {
    console.log('Serveur arrêté proprement');
    process.exit(0);
  });
});
```

**Node vs Deno vs Bun** — Deno (créé par l'auteur original de Node) : sécurité par défaut (permissions explicites), TypeScript natif. Bun : runtime tout-en-un (bundler, testeur, package manager inclus), très rapide, compatible API Node en grande partie. Node reste le standard de facto en entreprise par maturité d'écosystème ; les alternatives valent le coup d'œil pour de nouveaux projets.

## 6. Commandes / syntaxe à connaître

```bash
node app.js                  # exécuter un fichier
node --watch app.js          # relance auto au changement de fichier
node --inspect app.js        # debugger (chrome://inspect)
node --test                  # runner de tests intégré (Node 18+)

npm init -y                  # créer un package.json par défaut
npm install <pkg>            # ajouter une dépendance
npm install -D <pkg>         # ajouter une dépendance de dev
npm ci                       # install strict depuis le lockfile (CI/prod)
npm run <script>             # exécuter un script défini dans package.json
npm outdated / npm audit     # état des dépendances

npx <pkg>                    # exécuter un package sans l'installer globalement
NODE_ENV=production node app.js  # variable d'environnement inline
```

## 7. Exercices

Énoncés dans `exercices/`, corrections séparées dans `solutions/` — à ne consulter qu'après avoir essayé.

- **Niveau 1 — Bases** : `exercices/niveau-1.md`
- **Niveau 2 — Intermédiaire** : `exercices/niveau-2.md`
- **Niveau 3 — Avancé** : `exercices/niveau-3.md`

## 8. Mini-projet

**API REST "Todo list" en Node natif** (sans framework) :

- Serveur `http` natif avec routes `GET /todos`, `POST /todos`, `PATCH /todos/:id`, `DELETE /todos/:id`.
- Persistance simple dans un fichier JSON (via `fs/promises`), lu/écrit à chaque opération.
- Configuration (port) via variable d'environnement (`dotenv`).
- Gestion d'erreurs propre : corps JSON invalide → 400, route inconnue → 404, erreur serveur → 500 avec log.
- Arrêt propre (`SIGTERM`) qui termine les requêtes en cours avant de quitter.

Livrable : un projet Node avec `package.json`, `src/index.js`, `.env.example`, README d'utilisation minimal.

## Checklist

- [ ] Comprendre les fondamentaux (event loop, modules, npm)
- [ ] Savoir créer un projet Node (package.json, scripts)
- [ ] Maîtriser la syntaxe principale (fs, http, path, process)
- [ ] Comprendre les concepts importants (event loop détaillé, streams, EventEmitter, erreurs async)
- [ ] Savoir debugger (`--inspect`, DevTools, logs)
- [ ] Connaître les bonnes pratiques (async plutôt que sync, `npm ci`, secrets en env)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (cluster/worker_threads, mémoire, graceful shutdown)

## 10. Ressources

- [Documentation officielle Node.js](https://nodejs.org/docs/latest/api/) — référence API complète, à privilégier systématiquement
- [Node.js — Event Loop, Timers, and process.nextTick()](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick) — explication officielle de l'event loop
- [roadmap.sh — Node.js](https://roadmap.sh/nodejs) — parcours structuré, bon complément pour visualiser l'écosystème
- [npm Docs](https://docs.npmjs.com/) — référence officielle npm
