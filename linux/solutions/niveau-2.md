# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```bash
#!/bin/bash
set -euo pipefail

usage=$(df / | tail -1 | awk '{print $5}' | tr -d '%')

if [ "$usage" -gt 80 ]; then
    echo "ATTENTION : utilisation disque à ${usage}%"
fi
```

## Exercice 2

```cron
0 * * * * /usr/local/bin/check_disk.sh >> /var/log/check_disk.log 2>&1
```

## Exercice 3

```ini
# /etc/systemd/system/check_disk.service
[Unit]
Description=Vérification de l'espace disque

[Service]
Type=oneshot
ExecStart=/usr/local/bin/check_disk.sh
```

## Exercice 4

```bash
journalctl -u mon-app -f          # suivi en temps réel
journalctl -u mon-app --since "2 hours ago"   # logs des 2 dernières heures
```

## Exercice 5

```bash
ss -tulpn | grep 8080     # vérifie qu'un processus écoute sur le port 8080
curl -v http://localhost:8080/    # teste manuellement la réponse HTTP
```
