# Solutions — Niveau 3 (Avancé)

## Exercice 1

Un service qui ignore `SIGTERM` empêche un arrêt propre lors d'un déploiement (rolling restart, mise à jour) : l'orchestrateur ou le script de déploiement envoie SIGTERM et attend que le processus se termine de lui-même, mais si le processus l'ignore, il continue de tourner et peut bloquer le remplacement de version ou laisser des connexions en cours mal fermées. En dernier recours, après un court délai sans réponse, on utilise `kill -9 PID` (SIGKILL) pour forcer l'arrêt immédiat.

## Exercice 2

```bash
strace -p 4321
```

Dans la sortie, on cherche l'appel système sur lequel le processus reste bloqué en boucle ou en attente (ex. `read()`/`recv()` sur un socket sans données, `futex()` en attente d'un verrou, ou des appels répétés à un fichier/ressource indisponible) — cela indique généralement une attente réseau, un deadlock, ou une ressource manquante.

## Exercice 3

Un **namespace** isole la *vue* qu'un processus a du système : namespace PID (le processus voit son propre arbre de processus, isolé de l'hôte), namespace réseau (sa propre interface réseau et ses propres ports), namespace filesystem (son propre point de montage racine). Un **cgroup** (control group) limite et comptabilise les *ressources* consommées par un groupe de processus (CPU, mémoire, I/O), indépendamment de ce qu'ils voient. Docker combine les deux : les namespaces donnent à un conteneur l'illusion d'être une machine isolée, les cgroups empêchent un conteneur de consommer toutes les ressources de l'hôte — sans ces deux mécanismes noyau, il n'y aurait ni isolation ni limitation, donc pas de conteneurisation au sens Docker.

## Exercice 4

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

## Exercice 5

```
/var/log/mon-app.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```

Rotation quotidienne (`daily`), conservation de 7 fichiers archivés (`rotate 7`), compression des anciens fichiers (`compress`) pour économiser l'espace disque, sans erreur si le fichier est absent (`missingok`) ni rotation d'un fichier vide (`notifempty`).
