# Niveau 3 — Solutions

## 3.1 — Détection de fuite mémoire

```php
<?php
$leak = [];

while (frankenphp_handle_request(function () use (&$leak) {
    $leak[] = str_repeat('x', 1_000_000); // ~1 Mo accumulé à chaque requête, jamais libéré
    error_log('Mémoire utilisée : ' . memory_get_usage(true) . ' octets, ' . count($leak) . ' entrées');
    echo "OK\n";
})) {
}
```

En observant les logs sur plusieurs requêtes successives, `memory_get_usage()` croît de façon monotone : c'est la signature d'une fuite mémoire, invisible en mode classic/PHP-FPM où le process (et donc la mémoire) est recyclé à chaque requête.

```caddyfile
{
    frankenphp {
        worker {
            file ./public/worker.php
            max_requests 500
        }
    }
}
```

`max_requests` force FrankenPHP à redémarrer le worker (et donc à repartir d'une mémoire propre) après un nombre de requêtes donné. **Ce n'est pas une correction du bug** — le tableau `$leak` continue de grossir jusqu'au redémarrage — mais un filet de sécurité qui borne les dégâts en production le temps de corriger la vraie cause (ici : vider ou ne jamais alimenter `$leak` sans limite).

## 3.2 — Migration classic → worker

Démarche : lister tout ce qui est déclaré `static` ou en variable globale, toute connexion (PDO, Redis) ouverte hors d'une fonction rappelée à chaque requête, tout singleton maison stockant un état mutable. Pour chaque cas :
- Connexions DB : les réutiliser (bonne pratique en worker, évite de rouvrir une connexion à chaque requête) mais s'assurer qu'aucune transaction ou état de connexion ne reste "à moitié fait" entre deux requêtes (toujours committer/rollback avant la fin du traitement d'une requête).
- Variables statiques métier (compteurs, caches maison) : soit les accepter comme cache volontaire borné (avec une taille max claire), soit les déplacer vers un store externe (Redis) si elles doivent être partagées entre plusieurs workers/process.
- Singletons avec état mutable : les rendre "stateless" (aucune donnée métier stockée dans l'instance) ou les réinitialiser explicitement en tout début de traitement de requête.

## 3.3 — Décision d'architecture (exemple de trame ADR)

**Contexte** : application Symfony existante, montée en charge nécessaire sur les endpoints API à fort trafic.

**Options considérées** : rester en PHP-FPM classique, migrer entièrement vers FrankenPHP worker, migration partielle (worker uniquement sur certains endpoints critiques via une infrastructure séparée).

**Recommandation** : passer en mode worker apporte un bénéfice réel quand (a) le bootstrap du framework est une part significative du temps de réponse (beaucoup de services/configuration à charger), (b) le code applicatif a été audité et ne dépend d'aucun état global mutable non maîtrisé, (c) l'équipe peut mettre en place un monitoring mémoire/temps de réponse par requête. Le risque dépasse le bénéfice quand l'application contient du code legacy avec des effets de bord globaux non documentés, ou quand l'équipe n'a pas la capacité de monitorer finement le comportement en production après bascule — dans ce cas, rester en mode classic (déjà plus simple à opérer que Nginx+PHP-FPM) est le choix le plus sûr dans un premier temps, avec une migration vers le mode worker en étape ultérieure une fois l'audit fait.
