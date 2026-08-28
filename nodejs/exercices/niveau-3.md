# Node.js — Exercices niveau 3 (avancé)

## Exercice 3.1 — Serveur avec `cluster`

Transforme un serveur HTTP simple pour qu'il utilise le module `cluster` et fasse tourner un worker process par cœur CPU disponible (`os.cpus().length`). Vérifie (via un endpoint qui renvoie `process.pid`) que les requêtes successives sont bien servies par des process différents.

## Exercice 3.2 — Détecter une fuite mémoire

Écris volontairement un script qui accumule des objets dans un tableau global à chaque appel d'une fonction, sans jamais le vider (fuite mémoire simulée). Utilise `node --inspect` et l'onglet Memory de Chrome DevTools pour observer la croissance du heap sur plusieurs appels, puis corrige la fuite.

## Exercice 3.3 — Graceful shutdown

Sur un serveur HTTP qui simule une requête lente (`setTimeout` de 3s avant de répondre), implémente un handler `SIGTERM` qui : cesse d'accepter de nouvelles connexions, attend que les requêtes en cours se terminent, puis quitte le process. Teste en envoyant `Ctrl+C` (ou `kill`) pendant qu'une requête est en cours.

## Exercice 3.4 — Worker thread pour un calcul lourd

Écris une fonction de calcul volontairement lourde (ex. une boucle de plusieurs centaines de millions d'itérations). Montre d'abord qu'elle bloque l'event loop (un autre endpoint HTTP devient injoignable pendant le calcul). Puis déporte-la dans un `worker_threads` et montre que le serveur reste réactif pendant le calcul.

## Exercice 3.5 — Rate limiter maison

Implémente un middleware (fonction qui s'insère avant le handler de route) qui limite chaque IP à 5 requêtes par 10 secondes sur un serveur `http` natif, en stockant les compteurs en mémoire (`Map`). Renvoie un `429 Too Many Requests` en cas de dépassement.
