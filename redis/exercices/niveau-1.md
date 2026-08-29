# Exercices Redis — Niveau 1 (Bases)

## Exercice 1 — Set/Get avec expiration

Stocke la clé `session:xyz` avec la valeur `"user42"` et une expiration de 1800 secondes. Vérifie le TTL restant avec la commande appropriée.

## Exercice 2 — Compteur atomique

Initialise `page:contact:views` à 0, puis incrémente-le de 1, puis de 10 d'un coup, en une seule commande à chaque fois.

## Exercice 3 — Hash

Stocke un utilisateur `user:5` avec les champs `name`, `email`, `role` via une seule commande `HSET`, puis récupère uniquement le champ `email`.

## Exercice 4 — Liste comme file d'attente

Ajoute trois emails à une file `queue:notifications` (dans l'ordre `email-a`, `email-b`, `email-c`), puis retire et affiche le plus ancien.

## Exercice 5 — Sorted Set

Ajoute trois joueurs à un classement `leaderboard:game1` avec leurs scores (`alice`: 300, `bob`: 450, `carol`: 200), puis affiche le classement du meilleur au moins bon avec les scores.
