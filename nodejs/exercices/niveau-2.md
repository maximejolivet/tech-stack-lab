# Node.js — Exercices niveau 2 (intermédiaire)

## Exercice 2.1 — EventEmitter personnalisé

Crée une classe `Panier` qui hérite de `EventEmitter`, avec une méthode `ajouter(produit)` qui émet un événement `produit:ajoute` transportant le produit. Abonne un listener qui logue `"Ajouté : <produit>"` à chaque émission.

## Exercice 2.2 — Petit serveur avec routing JSON

Avec `http` natif (sans framework), crée un serveur exposant :
- `GET /users` → renvoie un tableau JSON en dur
- `POST /users` → lit le corps de la requête (JSON), et le renvoie tel quel avec un status 201

Indice : sur `POST`, il faut écouter les événements `data` et `end` du `req` (qui est un stream) pour reconstituer le corps.

## Exercice 2.3 — Compter les lignes avec un stream

Écris un script qui compte le nombre de lignes d'un gros fichier texte **sans le charger entièrement en mémoire**, en utilisant `fs.createReadStream`.

## Exercice 2.4 — Gestion d'erreur async

Écris une fonction `async` `recupererDonnees()` qui simule un échec (`throw new Error('Timeout')` après un `setTimeout`). Appelle-la avec un `try`/`catch`, et ajoute en plus un handler global `process.on('unhandledRejection', ...)` pour capturer le cas où quelqu'un oublierait le `try`/`catch`.

## Exercice 2.5 — Configuration via `.env`

Crée un fichier `.env` avec une variable `PORT=4000`, charge-le avec le package `dotenv`, et utilise `process.env.PORT` (avec une valeur par défaut si absente) pour démarrer un serveur `http` sur ce port. N'oublie pas d'ajouter `.env` à un `.gitignore`.
