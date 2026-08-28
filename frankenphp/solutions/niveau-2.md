# Niveau 2 — Solutions

## 2.1 — Mode worker minimal

```php
<?php
// public/worker.php
$count = 0;

while (frankenphp_handle_request(function () use (&$count) {
    $count++;
    header('Content-Type: text/plain');
    echo "Requête n°{$count} depuis le démarrage du worker\n";
})) {
    // la boucle continue tant que le worker est actif
}
```

```caddyfile
{
    frankenphp {
        worker ./public/worker.php
    }
}

localhost {
    root * public/
    php_server
}
```

Chaque requête successive affiche un compteur incrémenté : `$count` a survécu entre les requêtes, ce qui ne se produirait jamais en mode classic ou en PHP-FPM (où chaque requête repart d'un `$count = 0`).

## 2.2 — Piège de l'état partagé

```php
<?php
// Bug : $lastUser global jamais réinitialisé entre requêtes
$lastUser = null;

while (frankenphp_handle_request(function () use (&$lastUser) {
    $currentUser = $_GET['user'] ?? 'inconnu';
    echo "Utilisateur actuel : {$currentUser}\n";
    echo "Dernier utilisateur vu (BUG, ne devrait pas fuiter) : {$lastUser}\n";
    $lastUser = $currentUser; // fuite : partagé entre TOUTES les requêtes suivantes
})) {
}
```

Une requête `?user=alice` puis `?user=bob` : la deuxième affiche `alice` comme "dernier utilisateur vu", alors que ces deux requêtes proviennent potentiellement de deux clients complètement différents.

**Correction** : ne jamais stocker d'information par-requête dans une variable qui survit au worker. Si un historique par utilisateur est nécessaire, le stocker côté client (session/cookie) ou dans un store externe (Redis), jamais dans une variable PHP globale du worker.

```php
while (frankenphp_handle_request(function () {
    $currentUser = $_GET['user'] ?? 'inconnu'; // variable locale à la closure, recréée à chaque requête
    echo "Utilisateur actuel : {$currentUser}\n";
})) {
}
```

## 2.3 — Intégration framework

Symfony Runtime et Laravel Octane réinitialisent le conteneur de services (ou le "kernel") à chaque requête entrante en mode worker : les services applicatifs sont reconstruits (ou proprement remis à leur état initial) plutôt que réutilisés tels quels d'une requête à l'autre. Cela reproduit l'isolation du mode classic pour le code applicatif standard du framework — le risque de fuite d'état ne concerne alors que le code custom qui contourne ce cycle (variables globales/statiques manuelles, singletons maison).
