# Solutions — Niveau 3 (Avancé)

## Exercice 1

Utiliser le Cache Tag générique de liste `node_list` (ou plus précis, `node_list:event` si le module le définit) dans le tableau de rendu :

```php
return [
    '#cache' => ['tags' => ['node_list']],
    // ...
];
```

`node_list` est invalidé automatiquement par Drupal core à chaque création, modification ou suppression d'un node — le cache de l'endpoint se retrouve donc invalidé exactement quand nécessaire, sans purge manuelle ni durée d'expiration arbitraire.

## Exercice 2

```php
public function upcomingEvents(): JsonResponse {
    $nids = \Drupal::entityTypeManager()->getStorage('node')->getQuery()
        ->condition('type', 'event')
        ->condition('status', 1)
        ->sort('field_date', 'ASC')
        ->accessCheck(true)
        ->execute();

    $nodes = \Drupal\node\Entity\Node::loadMultiple($nids);
    $events = array_map(fn($node) => [
        'title' => $node->getTitle(),
        'date' => $node->get('field_date')->value,
        'link' => $node->toUrl()->setAbsolute()->toString(),
    ], $nodes);

    return new JsonResponse(array_values($events));
}
```

## Exercice 3

```php
class EventRequestSubscriber implements EventSubscriberInterface {
    public static function getSubscribedEvents(): array {
        return [KernelEvents::REQUEST => 'onRequest'];
    }

    public function onRequest(RequestEvent $event) {
        $path = $event->getRequest()->getPathInfo();
        if (str_starts_with($path, '/evenements')) {
            \Drupal::logger('mon_module')->info('Requête vers @path', ['@path' => $path]);
        }
    }
}
```

Un Event Subscriber s'accroche à un événement Symfony typé (ici `KernelEvents::REQUEST`, déclenché à chaque requête HTTP dans le cycle de vie du kernel), avec une classe explicite et découvrable via le conteneur de services — contrairement à un `hook_*` procédural, qui repose sur une convention de nommage de fonction globale scannée par Drupal, hérité de l'architecture pré-Symfony du cœur.

## Exercice 4

Trois familles de plugins Migrate :
- **Source** (`source`) : d'où viennent les données — ici un plugin `csv` qui lit le fichier et expose chaque ligne comme une "ligne source" (`title`, `date`, `location`).
- **Process** (`process`) : comment chaque champ source est transformé avant d'être écrit — ex. mapper `title` → `title`, convertir le format de `date` si nécessaire, générer un slug.
- **Destination** (`destination`) : où écrire le résultat — ici `entity:node` avec le bundle `event`, qui crée un node par ligne traitée.

La migration se déclare en YAML reliant ces trois plugins ; elle s'exécute ensuite via `drush migrate:import`, et reste ré-exécutable/annulable (`migrate:rollback`) sans dupliquer les données déjà importées.

## Exercice 5

BigPipe envoie immédiatement au navigateur le HTML de la page qui **est** cacheable (structure commune, contenu public), avec des espaces réservés pour les portions non-cacheables, puis streame ces portions ("pipe") au fur et à mesure qu'elles sont calculées côté serveur, dans la même réponse HTTP. Un élément de rendu devient éligible à ce traitement dès qu'il est déclaré via `#lazy_builder` (callback exécuté à la demande, hors du rendu principal mis en cache) plutôt que calculé directement dans l'arbre de rendu — c'est ce qui permet à un bloc "Bonjour {utilisateur}" de ne pas empêcher la mise en cache du reste de la page pour les autres visiteurs.
