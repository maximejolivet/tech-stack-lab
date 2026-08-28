# Niveau 1 — Solutions

## 1.1 — Premier démarrage

```bash
mkdir public && echo '<?php phpinfo();' > public/index.php
frankenphp php-server --root public/
```

Dans la sortie de `phpinfo()`, la ligne `Server API` indique un SAPI lié à FrankenPHP (embed SAPI), différent de `cgi-fcgi` (PHP-FPM classique).

## 1.2 — Caddyfile minimal

```caddyfile
{
    frankenphp
}

localhost {
    root * public/
    php_server
}
```

```bash
frankenphp run --config Caddyfile
```

Pas de directive `worker` : les requêtes sont traitées en mode classic, chaque requête réexécute `index.php` depuis le début.

## 1.3 — Comparaison rapide

En mode classic, FrankenPHP se comporte fonctionnellement comme PHP-FPM (nouveau contexte d'exécution à chaque requête, pas d'état partagé). Ce qui change, c'est l'infrastructure : un seul binaire/process fait à la fois le rôle du serveur web (Nginx) et du gestionnaire PHP (PHP-FPM), avec HTTPS automatique et une configuration plus simple (Caddyfile au lieu de deux fichiers de config séparés). Pour un développeur qui reste en mode classic, le code applicatif PHP n'a rien à changer.
