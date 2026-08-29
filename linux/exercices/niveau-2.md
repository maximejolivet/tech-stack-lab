# Exercices Linux — Niveau 2 (Intermédiaire)

## Exercice 1 — Script Bash robuste

Écris un script `check_disk.sh` qui affiche un avertissement si l'utilisation du disque racine (`/`) dépasse 80%. Le script doit commencer par les options de robustesse recommandées (`set -euo pipefail`).

## Exercice 2 — Tâche cron

Écris la ligne crontab qui exécute le script `check_disk.sh` toutes les heures, en redirigeant sa sortie standard et ses erreurs vers `/var/log/check_disk.log`.

## Exercice 3 — Unité systemd

Écris un fichier d'unité systemd `check_disk.service` de type `oneshot` qui exécute `/usr/local/bin/check_disk.sh`, avec une description claire.

## Exercice 4 — Logs d'un service

Explique la commande à utiliser pour suivre en temps réel les logs d'un service nommé `mon-app`, et celle pour n'afficher que les logs des dernières 2 heures.

## Exercice 5 — Diagnostic réseau

Un service web ne répond pas sur le port 8080. Donne la commande pour vérifier si un processus écoute bien sur ce port, et celle pour tester manuellement la réponse HTTP avec `curl`.
