# Solutions — Niveau 1 (Bases)

## Exercice 1

```bash
cd /var/log
pwd
cd -
```

## Exercice 2

```bash
touch deploy.sh
chmod 750 deploy.sh
ls -l deploy.sh
# -rwxr-x--- 1 user group ... deploy.sh
```

## Exercice 3

```bash
ps aux | grep ssh
```

`kill -15` (SIGTERM) demande un arrêt propre : le processus reçoit le signal et peut nettoyer ses ressources (fermer des connexions, sauvegarder un état) avant de quitter. `kill -9` (SIGKILL) force l'arrêt immédiat sans laisser au processus la moindre chance de réagir — à utiliser seulement si `kill -15` n'a pas fonctionné.

## Exercice 4

```bash
grep -c "ERROR" access.log
echo "Fin de rapport" >> access.log
```

## Exercice 5

```bash
sudo apt update
sudo apt install htop
sudo useradd -m bob
sudo usermod -aG sudo bob
```
