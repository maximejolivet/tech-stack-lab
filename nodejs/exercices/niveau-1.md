# Node.js — Exercices niveau 1 (bases)

## Exercice 1.1 — Lire un fichier

Écris un script `lire.js` qui lit un fichier `data.txt` (crée-le avec un peu de texte) de façon **asynchrone** avec `fs/promises`, et affiche son contenu dans la console. Gère le cas où le fichier n'existe pas (message clair, pas de crash brut).

## Exercice 1.2 — Serveur HTTP minimal

Avec le module `http` natif, crée un serveur qui répond `"Hello Node"` en texte brut sur `GET /`, et un `404` en JSON (`{ "error": "Not found" }`) pour toute autre route.

## Exercice 1.3 — Arguments de la ligne de commande

Écris un script `salut.js` exécutable ainsi : `node salut.js Max` et qui affiche `Bonjour Max !`. Utilise `process.argv` (attention à l'index : les deux premiers éléments ne sont pas tes arguments).

## Exercice 1.4 — CommonJS vers ESM

Convertis ce fichier CommonJS en ESM (import/export), et explique ce qu'il faut changer dans `package.json` pour que ça fonctionne :

```js
const path = require('path');
function joinPaths(a, b) {
  return path.join(a, b);
}
module.exports = { joinPaths };
```

## Exercice 1.5 — Script npm

Dans un `package.json`, ajoute un script `dev` qui lance `app.js` avec le flag `--watch`, et un script `start` qui le lance normalement. Donne les commandes `npm` pour exécuter chacun.
