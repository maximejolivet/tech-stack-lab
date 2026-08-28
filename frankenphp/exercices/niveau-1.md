# Niveau 1 — Bases

## Exercice 1.1 — Premier démarrage

Démarre FrankenPHP (via Docker ou le binaire standalone) et sers un simple `index.php` qui affiche `phpinfo()`. Vérifie dans la sortie que le SAPI utilisé est bien lié à FrankenPHP.

## Exercice 1.2 — Caddyfile minimal

Écris un `Caddyfile` qui sert le contenu du dossier `public/` sur `localhost`, en mode classic (pas de worker). Démarre FrankenPHP avec ce fichier.

## Exercice 1.3 — Comparaison rapide

Sans coder, explique en quelques phrases : quelle est la différence fondamentale entre PHP-FPM classique et FrankenPHP en mode classic (pas worker) ? Que change FrankenPHP concrètement pour un développeur qui ne fait que du mode classic ?
