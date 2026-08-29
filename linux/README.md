# Linux

## 1. Introduction

Linux est un système d'exploitation open source de type Unix, omniprésent comme environnement d'exécution serveur (cloud, conteneurs, CI/CD). Ce dossier couvre l'usage en ligne de commande (shell Bash) nécessaire pour développer, déployer et diagnostiquer une application — pas l'administration système complète d'un poste de travail.

**À quoi sert-il ?**
- Exécuter et exploiter des serveurs applicatifs, bases de données, pipelines CI/CD.
- Automatiser des tâches (scripts, cron) et diagnostiquer des problèmes en production (logs, processus, ressources).
- Servir de base à quasiment tous les outils modernes de déploiement : conteneurs (voir [`../docker/`](../docker/)), CI/CD, cloud.

**Où se situe-t-il dans une architecture web ?** La couche système sur laquelle tourne tout le reste : le serveur web, le runtime applicatif, la base de données, et les conteneurs eux-mêmes (un conteneur Docker partage le noyau Linux de son hôte).

**Avantages** : gratuit, stable, scriptable de bout en bout, standard de facto pour l'hébergement et les conteneurs, transparence totale sur ce qui s'exécute.
**Limites** : courbe d'apprentissage de la ligne de commande pour les habitués d'interfaces graphiques, fragmentation des distributions (Debian/Ubuntu vs RHEL/CentOS) avec des gestionnaires de paquets différents.

## 2. Prérequis

- Aucun prérequis technique strict — ce dossier peut être abordé tôt dans la roadmap.
- Accès à un terminal Linux (machine native, VM, WSL sous Windows, ou conteneur Docker `ubuntu`/`debian` pour s'entraîner sans risque).

## 3. Rappel des bases 🟢

### 01 - Navigation dans le système de fichiers

**Explication** — Le système de fichiers Linux est une arborescence unique enracinée à `/` (pas de lettres de lecteur comme sous Windows). Les commandes de base : `pwd` (afficher le répertoire courant), `cd` (se déplacer), `ls` (lister).

```bash
pwd                 # affiche le chemin absolu courant
cd /var/log         # se déplacer vers /var/log
ls -la              # lister tout, y compris fichiers cachés, en format détaillé
cd ~                # revenir au répertoire personnel de l'utilisateur
cd -                # revenir au répertoire précédent
```

**Bonne pratique** : préférer les chemins relatifs (`./`, `../`) dans les scripts portables, mais toujours vérifier avec `pwd` avant une commande destructive (`rm`, `mv`).

### 02 - Hiérarchie du système de fichiers (FHS)

**Explication** — Chaque répertoire racine a un rôle standardisé (Filesystem Hierarchy Standard) : `/etc` (configuration), `/var` (données variables : logs, cache), `/home` (répertoires utilisateurs), `/usr/bin` et `/bin` (exécutables), `/tmp` (fichiers temporaires, souvent vidé au redémarrage), `/opt` (logiciels tiers autonomes).

**Bonne pratique** : ne jamais stocker de données applicatives persistantes dans `/tmp` — rien ne garantit qu'elles survivront à un redémarrage.

### 03 - Permissions et propriété

**Explication** — Chaque fichier a un propriétaire, un groupe, et trois jeux de permissions (lecture/écriture/exécution) pour le propriétaire, le groupe, et les autres. `chmod` modifie les permissions, `chown` le propriétaire/groupe.

```bash
ls -l script.sh                  # ex: -rwxr-xr-- 1 alice devs 220 ...
chmod +x script.sh                # rendre exécutable pour tous
chmod 750 script.sh                # rwx pour le propriétaire, r-x pour le groupe, rien pour les autres
chown alice:devs script.sh         # changer propriétaire et groupe
```

**Erreur fréquente** : utiliser `chmod 777` pour "faire fonctionner" un script sans comprendre le problème réel — cela ouvre l'écriture et l'exécution à tout le monde, un risque de sécurité sur un serveur partagé.

**Bonne pratique** : accorder le minimum de permissions nécessaire (principe du moindre privilège) — voir [`../security/`](../security/).

### 04 - Gestion des processus

**Explication** — Chaque programme en cours d'exécution est un processus identifié par un PID. `ps` liste les processus, `top`/`htop` les affiche en temps réel avec leur consommation de ressources, `kill` envoie un signal à un processus.

```bash
ps aux | grep nginx      # lister les processus dont le nom contient "nginx"
top                        # vue en temps réel de l'usage CPU/mémoire
kill -15 1234               # demander l'arrêt propre du processus 1234 (SIGTERM)
kill -9 1234                 # forcer l'arrêt immédiat (SIGKILL, en dernier recours)
```

**Bonne pratique** : toujours essayer `kill -15` (arrêt propre, laisse le temps au processus de nettoyer/fermer ses connexions) avant `kill -9` (arrêt brutal, aucun nettoyage).

### 05 - Redirections et pipes

**Explication** — Le shell permet de rediriger les flux standard (`stdin`, `stdout`, `stderr`) et de chaîner des commandes : la sortie de l'une devient l'entrée de la suivante via `|`.

```bash
echo "log" > fichier.txt          # écrire (écrase le fichier existant)
echo "autre log" >> fichier.txt    # ajouter à la fin du fichier
cat access.log | grep "ERROR" | wc -l   # compter les lignes contenant "ERROR"
commande 2> erreurs.log             # rediriger uniquement stderr vers un fichier
```

**Erreur fréquente** : utiliser `>` au lieu de `>>` par réflexe et écraser un fichier de logs existant sans le vouloir.

### 06 - Gestion des paquets

**Explication** — Chaque distribution a son gestionnaire de paquets : `apt` (Debian/Ubuntu), `dnf`/`yum` (RHEL/Fedora), `pacman` (Arch). Il installe, met à jour et supprime des logiciels avec résolution automatique des dépendances.

```bash
sudo apt update              # rafraîchir la liste des paquets disponibles
sudo apt install nginx        # installer un paquet
sudo apt upgrade               # mettre à jour les paquets installés
sudo apt remove nginx           # désinstaller un paquet
```

**Bonne pratique** : toujours faire `apt update` avant un `install`/`upgrade` — sans ça, on installe potentiellement une version obsolète ou on rate un correctif de sécurité récent.

### 07 - Utilisateurs et groupes

**Explication** — Chaque utilisateur appartient à un ou plusieurs groupes, qui déterminent en partie ses permissions. `sudo` exécute une commande ponctuelle avec les privilèges d'un autre utilisateur (root par défaut).

```bash
sudo useradd -m -s /bin/bash alice   # créer un utilisateur avec home et shell bash
sudo usermod -aG docker alice          # ajouter alice au groupe "docker"
whoami                                   # afficher l'utilisateur courant
id alice                                  # afficher les groupes d'un utilisateur
```

**Erreur fréquente** : travailler en permanence connecté en tant que `root` plutôt qu'avec un utilisateur standard + `sudo` ponctuel — une erreur de commande a un impact bien plus large en étant root en continu.

## 4. Concepts intermédiaires 🟡

- **Scripting shell (Bash)** : automatiser une suite de commandes dans un fichier `.sh`, avec variables, conditions, boucles.

```bash
#!/bin/bash
set -euo pipefail   # arrêt immédiat en cas d'erreur, variable non définie, ou échec dans un pipe

for file in /var/log/*.log; do
    if grep -q "ERROR" "$file"; then
        echo "Erreurs trouvées dans $file"
    fi
done
```

**Bonne pratique** : commencer systématiquement un script par `set -euo pipefail` — sans ça, une commande qui échoue silencieusement peut laisser le script continuer dans un état incohérent.

- **Cron** : planificateur de tâches récurrentes, configuré via `crontab -e` (syntaxe `minute heure jour mois jour_semaine commande`).

```bash
# Exécuter un script de sauvegarde tous les jours à 2h du matin
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

- **systemd et services** : gestionnaire de services standard des distributions modernes. Une unité `.service` décrit comment démarrer, arrêter et superviser un processus démon.

```ini
# /etc/systemd/system/mon-app.service
[Unit]
Description=Mon application
After=network.target

[Service]
ExecStart=/usr/bin/node /opt/mon-app/server.js
Restart=on-failure
User=www-data

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload      # recharger la config après modification d'une unité
sudo systemctl enable --now mon-app   # activer au démarrage + démarrer immédiatement
sudo systemctl status mon-app          # vérifier l'état
```

- **Gestion des logs (journalctl)** : les services gérés par systemd journalisent via `journald`, consultable avec `journalctl -u mon-app -f` (suivi en temps réel, équivalent de `tail -f` mais centralisé).
- **Outils réseau** : `ss -tulpn` (ports en écoute, remplace `netstat`), `curl`/`wget` (requêtes HTTP en ligne de commande), `ping`/`traceroute` (diagnostic de connectivité).
- **Durcissement SSH** : désactiver l'authentification par mot de passe au profit des clés (`PasswordAuthentication no` dans `/etc/ssh/sshd_config`), désactiver la connexion root directe (`PermitRootLogin no`) — voir [`../security/`](../security/).

## 5. Concepts avancés 🟠🔴

- **Gestion fine des processus et signaux** : au-delà de `kill -9`, comprendre les signaux `SIGHUP` (rechargement de config sans redémarrage complet, utilisé par nginx), `SIGTERM` (arrêt propre attendu), et l'importance de gérer ces signaux dans une application conteneurisée (PID 1 dans un conteneur ne bénéficie pas du reaping automatique des processus zombies par défaut).
- **Analyse de performance** : `iostat`/`vmstat` (I/O disque, mémoire), `strace` (tracer les appels système d'un processus pour diagnostiquer un blocage), `lsof` (lister les fichiers/ports ouverts par un processus).
- **Namespaces et cgroups** : les deux mécanismes noyau qui rendent la conteneurisation possible — les namespaces isolent la vue qu'un processus a du système (PID, réseau, filesystem), les cgroups limitent et comptabilisent ses ressources (CPU, mémoire). Comprendre ces primitives éclaire ce que fait réellement Docker (voir [`../docker/`](../docker/)) sous le capot.
- **Automatisation d'infrastructure** : au-delà d'un script Bash ponctuel, des outils comme Ansible permettent de décrire et appliquer un état système de façon idempotente sur plusieurs machines — pertinent une fois la gestion manuelle de serveurs individuels maîtrisée.
- **Sécurité approfondie** : pare-feu (`ufw`/`iptables`), fail2ban (bannissement automatique après tentatives de connexion échouées répétées), audit des permissions SUID/SGID, principe du moindre privilège appliqué systématiquement aux services (chaque service tourne sous un utilisateur dédié, jamais root).
- **Haute disponibilité et supervision** : sondes de santé exploitées par un orchestrateur (voir [`../kubernetes/`](../kubernetes/)), rotation des logs (`logrotate`) pour éviter la saturation du disque, monitoring système (Prometheus node_exporter) pour l'observabilité.

## 6. Commandes / syntaxe à connaître

```bash
pwd; cd; ls -la                       # navigation
chmod 750 f; chown user:group f       # permissions
ps aux; top; kill -15 PID              # processus
cat f | grep motif | wc -l              # pipes
sudo apt update && sudo apt install x  # paquets (Debian/Ubuntu)
systemctl status/start/stop/enable x    # services
journalctl -u x -f                       # logs d'un service en temps réel
crontab -e                                # éditer les tâches planifiées
ss -tulpn                                  # ports en écoute
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Script de sauvegarde automatisé et supervisé**

Construire une chaîne complète d'automatisation système :
- Un script Bash `backup.sh` (avec `set -euo pipefail`) qui archive un répertoire donné en `.tar.gz` horodaté dans `/var/backups/`, et supprime les archives de plus de 7 jours.
- Une unité systemd de type `oneshot` (`backup.service`) qui exécute ce script.
- Une tâche cron (ou un timer systemd) qui déclenche ce service tous les jours à 3h du matin.
- Un journal des exécutions consultable via `journalctl -u backup.service`.

Bonus : ajouter un contrôle qui envoie un message (ex. `curl` vers un webhook) si le script échoue (`set -e` + trap sur erreur), et durcir l'accès SSH de la machine (clé uniquement, `PermitRootLogin no`).

## Checklist

- [ ] Comprendre les fondamentaux (arborescence, permissions, processus, pipes)
- [ ] Savoir naviguer et manipuler des fichiers en ligne de commande sans interface graphique
- [ ] Maîtriser la syntaxe principale (`chmod`/`chown`, `ps`/`kill`, redirections)
- [ ] Comprendre les concepts importants (scripting Bash, cron, systemd, logs)
- [ ] Savoir debugger (`journalctl`, `ps aux`, `ss -tulpn`, `strace`)
- [ ] Connaître les bonnes pratiques (moindre privilège, `set -euo pipefail`, éviter `chmod 777`)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (namespaces/cgroups, signaux, durcissement, supervision)

## 10. Ressources

- [The Linux Command Line (livre gratuit)](https://linuxcommand.org/tlcl.php) — référence complète et progressive.
- [Explainshell](https://explainshell.com/) — décompose n'importe quelle commande shell complexe.
- [systemd — documentation officielle Arch Wiki](https://wiki.archlinux.org/title/Systemd) — référence claire et à jour, applicable au-delà d'Arch.
- [roadmap.sh — Linux](https://roadmap.sh/linux) — vue d'ensemble structurée des compétences à couvrir.
