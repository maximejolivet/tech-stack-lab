# Exercices Linux — Niveau 3 (Avancé)

## Exercice 1 — Signaux et arrêt propre

Explique pourquoi un service qui ignore `SIGTERM` pose problème lors d'un déploiement, et quelle commande utiliser en dernier recours si le processus ne s'arrête toujours pas après quelques secondes.

## Exercice 2 — strace

Un script semble bloqué sans erreur visible. Donne la commande `strace` permettant d'observer les appels système effectués par le processus de PID 4321, et explique ce que tu chercherais dans la sortie.

## Exercice 3 — Namespaces et cgroups

Explique en 3-4 phrases la différence entre un namespace et un cgroup, et pourquoi ces deux mécanismes noyau sont indispensables au fonctionnement de Docker.

## Exercice 4 — Pare-feu minimal

Avec `ufw`, écris les commandes pour autoriser uniquement le trafic SSH (port 22) et HTTP/HTTPS (ports 80 et 443), puis activer le pare-feu.

## Exercice 5 — Rotation des logs

Un service écrit des logs sans limite dans `/var/log/mon-app.log`, au risque de saturer le disque. Décris la configuration `logrotate` minimale (fréquence, nombre de fichiers conservés, compression) pour éviter ce problème.
