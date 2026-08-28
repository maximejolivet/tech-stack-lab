# FrankenPHP

## 1. Introduction

FrankenPHP est un serveur d'application PHP moderne, écrit en Go et basé sur [Caddy](https://caddyserver.com/). Il embarque l'interpréteur PHP directement dans son binaire : une alternative au duo classique Nginx + PHP-FPM. Ce dossier couvre uniquement le runtime — PHP natif est dans [`../php/`](../php/), les frameworks qui tournent dessus dans [`../symfony/`](../symfony/) et [`../laravel/`](../laravel/).

**À quoi sert-il ?**
- Simplifier le déploiement d'une app PHP : un seul binaire (ou une seule image Docker), pas de configuration Nginx séparée.
- Exécuter du PHP en mode "worker" : le script reste chargé en mémoire entre les requêtes, au lieu d'être ré-interprété à chaque fois.
- Fournir HTTPS automatique (certificats Let's Encrypt gérés par Caddy) et HTTP/3 sans configuration additionnelle.

**Où se situe-t-il dans une architecture web ?**
Il remplace la couche serveur web + gestionnaire de processus PHP (Nginx/Apache + PHP-FPM). Il reçoit directement les requêtes HTTP et exécute le code PHP de l'application (Symfony, Laravel, ou du PHP natif).

**Avantages**
- Déploiement radicalement plus simple (un binaire ou une image Docker officielle).
- Mode worker : gains de performance significatifs (pas de bootstrap complet du framework à chaque requête).
- HTTPS/HTTP2/HTTP3 automatiques, intégration native avec Symfony (Symfony Runtime) et Laravel Octane.

**Limites**
- Le mode worker change le modèle mental "share-nothing" de PHP : l'état global doit être géré avec précaution (voir section avancée).
- Écosystème plus jeune que Nginx + PHP-FPM : moins de retours d'expérience en production à très grande échelle, moins d'outillage tiers mature.
- Certaines librairies/codebases legacy avec effets de bord globaux ne sont pas compatibles telles quelles avec le mode worker.

## 2. Prérequis

- PHP natif maîtrisé (voir [`../php/`](../php/)) et idéalement une base sur Symfony ou Laravel.
- Comprendre la différence entre le modèle "share-nothing" classique de PHP-FPM (chaque requête part d'un état propre) et un runtime persistant (Node.js par exemple, voir [`../nodejs/`](../nodejs/)) — c'est la clé pour comprendre les pièges du mode worker.
- Docker installé, utile pour tester rapidement sans configuration locale.

## 3. Rappel des bases 🟢

### 01 - Qu'est-ce que FrankenPHP

**Explication** — FrankenPHP embarque l'interpréteur PHP (via l'extension Zend Engine liée en Go) dans le serveur Caddy. Il peut fonctionner en mode **classic** (comportement proche de PHP-FPM : un processus par requête, aucun état partagé) ou en mode **worker** (le script d'entrée reste en mémoire et traite les requêtes en boucle).

**Cas d'usage** : remplacer Nginx + PHP-FPM pour une app Symfony/Laravel, en dev comme en prod.

**Bonne pratique** : commencer en mode classic pour une migration, ne passer en mode worker qu'une fois l'app validée comme compatible (pas d'état global mutable non maîtrisé).

### 02 - Installation et démarrage rapide

**Explication** — Binaire standalone téléchargeable, ou image Docker officielle `dunglas/frankenphp`.

```bash
# Binaire standalone
frankenphp php-server --root public/

# Docker
docker run -v $PWD:/app -p 80:80 -p 443:443 dunglas/frankenphp
```

**Erreur fréquente** : oublier que le port 443 nécessite un domaine valide pour le HTTPS automatique en production ; en local, FrankenPHP génère un certificat auto-signé (accepter l'avertissement du navigateur ou utiliser `localhost`).

### 03 - Le Caddyfile

**Explication** — FrankenPHP est configuré via un `Caddyfile`, la syntaxe de configuration de Caddy, étendue avec une directive `php_server`.

```caddyfile
{
    frankenphp
}

localhost {
    root * public/
    php_server
}
```

**Bonne pratique** : garder le `Caddyfile` versionné avec le projet (comme un `nginx.conf`), il documente la configuration serveur au même titre que le code applicatif.

### 04 - Mode classic vs mode worker

**Explication** — En mode classic, chaque requête exécute le script PHP depuis le début (comme PHP-FPM). En mode worker, un script "boot" (souvent `public/frankenphp-worker.php` fourni par Symfony Runtime ou Laravel Octane) reste chargé et traite les requêtes dans une boucle, sans recharger le framework à chaque fois.

```php
// Exemple simplifié du principe d'un worker
require __DIR__ . '/vendor/autoload.php';
$app = require __DIR__ . '/src/bootstrap.php'; // bootstrap une seule fois

while (frankenphp_handle_request($handler ?? null)) {
    $app->handleRequest(); // rejoué à chaque requête, état applicatif réinitialisé par le framework
}
```

**Cas d'usage** : mode worker pour maximiser le débit d'une API à fort trafic ; mode classic pour un site simple ou une migration progressive.

**Erreur fréquente** : croire que le mode worker est un pur "plus rapide, sans contrepartie" — il change le modèle d'exécution et demande de vérifier que le framework/l'app ne fuit pas d'état entre requêtes.

## 4. Concepts intermédiaires 🟡

- **État partagé en mode worker** : les variables statiques, singletons mal conçus ou connexions non fermées peuvent "fuiter" d'une requête à l'autre puisque le processus PHP n'est plus tué à chaque fin de requête. Symfony et Laravel gèrent cela nativement (réinitialisation du conteneur de services entre requêtes via leurs intégrations officielles), mais du code custom avec des variables globales/statiques doit être audité.
- **Intégration Symfony** : via le composant [Symfony Runtime](https://symfony.com/doc/current/runtime.html#frankenphp), qui gère automatiquement le cycle worker et réinitialise le kernel entre les requêtes.
- **Intégration Laravel** : via [Laravel Octane](https://laravel.com/docs/octane), qui supporte FrankenPHP comme un des runtimes possibles (au même titre que Swoole/RoadRunner).
- **HTTPS automatique** : Caddy obtient et renouvelle automatiquement les certificats Let's Encrypt pour un nom de domaine public — aucune configuration Certbot à gérer.
- **Comparaison avec PHP-FPM classique** : PHP-FPM = process manager séparé du serveur web (Nginx en frontal) ; FrankenPHP = serveur web + runtime PHP dans un seul processus. Moins de pièces mobiles, mais moins de "chacun son rôle" pour du troubleshooting fin.

## 5. Concepts avancés 🟠🔴

- **Gestion de la mémoire en mode worker** : sans redémarrage de processus entre requêtes, une fuite mémoire (ressource non libérée, cache applicatif non borné) s'accumule au fil du temps au lieu d'être nettoyée automatiquement — surveiller la consommation mémoire est bien plus critique qu'en PHP-FPM classique. FrankenPHP propose un nombre maximum de requêtes par worker avant redémarrage (`max_requests`) comme filet de sécurité.
- **Scalabilité et production** : nombre de workers à dimensionner selon les cœurs CPU disponibles (pas un par requête concurrente comme PHP-FPM, mais un pool fixe qui traite les requêtes en boucle) ; déploiement typique via l'image Docker officielle derrière un load balancer.
- **Monitoring** : surveiller le temps de traitement par requête (une lenteur dans un worker bloque les requêtes suivantes de ce worker) et la mémoire résidente dans le temps (détection de fuite).
- **Quand ne pas utiliser le mode worker** : code legacy s'appuyant sur des variables globales mutables, extensions PHP non thread-safe, ou applications où chaque requête doit repartir d'un état garanti vierge sans audit préalable possible.

## 6. Commandes / syntaxe à connaître

```bash
frankenphp php-server --root public/         # démarrage rapide, mode classic
frankenphp run                                # démarrage avec Caddyfile personnalisé
frankenphp version                             # version installée

docker run -v $PWD:/app -p 80:80 -p 443:443 dunglas/frankenphp   # via Docker
```

```caddyfile
# Caddyfile minimal
{
    frankenphp {
        worker ./public/frankenphp-worker.php
    }
}

localhost {
    root * public/
    php_server
}
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Migrer une petite app PHP natif vers FrankenPHP, en mode classic puis worker**

- Partir du mini-projet CLI/HTTP simple d'un autre dossier (ou un petit routeur PHP natif fait maison), servi en PHP-FPM classique.
- Le faire tourner sous FrankenPHP en mode classic (Docker ou binaire), avec un `Caddyfile` minimal.
- Basculer en mode worker et identifier tout état global problématique (variable statique, connexion DB non fermée) en observant le comportement entre deux requêtes successives.
- Ajouter `max_requests` et documenter pourquoi cette limite est un filet de sécurité utile en worker.

Objectif : comprendre concrètement la différence entre les deux modes et les précautions qu'impose un runtime persistant.

## Checklist

- [ ] Comprendre les fondamentaux (serveur embarqué, mode classic vs worker)
- [ ] Savoir démarrer un projet avec FrankenPHP (binaire ou Docker)
- [ ] Maîtriser la syntaxe du Caddyfile de base
- [ ] Comprendre les concepts importants (intégration Symfony Runtime / Laravel Octane, HTTPS automatique)
- [ ] Savoir debugger un comportement lié à l'état partagé en mode worker
- [ ] Connaître les bonnes pratiques (quand utiliser worker vs classic, `max_requests`)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (gestion mémoire, scalabilité, monitoring)

## 10. Ressources

- [Documentation officielle FrankenPHP](https://frankenphp.dev/fr/docs/) — référence complète, installation, configuration.
- [Symfony Runtime — FrankenPHP](https://symfony.com/doc/current/runtime.html#frankenphp) — intégration officielle Symfony.
- [Laravel Octane](https://laravel.com/docs/octane) — intégration Laravel avec support FrankenPHP.
- [Documentation Caddy](https://caddyserver.com/docs/) — pour approfondir la configuration du serveur sous-jacent.
