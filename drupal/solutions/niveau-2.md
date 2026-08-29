# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```php
/**
 * @Block(
 *   id = "next_events_block",
 *   admin_label = @Translation("Prochains événements")
 * )
 */
class NextEventsBlock extends BlockBase {
    public function build(): array {
        return ['#markup' => $this->t('Prochains événements')];
    }
}
```

## Exercice 2

```yaml
# mon_module.routing.yml
mon_module.hello:
  path: '/annuaire/bonjour'
  defaults:
    _controller: '\Drupal\mon_module\Controller\HelloController::hello'
  requirements:
    _permission: 'access content'
```

```php
class HelloController extends ControllerBase {
    public function hello(): array {
        return ['#markup' => 'Bonjour !'];
    }
}
```

## Exercice 3

```yaml
# mon_module.services.yml
services:
  mon_module.event_helper:
    class: Drupal\mon_module\EventHelper
```

```php
class HelloController extends ControllerBase {
    public function __construct(private EventHelper $eventHelper) {}

    public static function create(ContainerInterface $container): static {
        return new static($container->get('mon_module.event_helper'));
    }

    public function hello(): array {
        return ['#markup' => 'Bonjour !'];
    }
}
```

## Exercice 4

```php
$nids = \Drupal::entityTypeManager()
    ->getStorage('node')
    ->getQuery()
    ->condition('type', 'event')
    ->condition('status', 1)
    ->sort('field_date', 'ASC')
    ->accessCheck(true)
    ->execute();
```

## Exercice 5

```yaml
# mon_module.permissions.yml
manage event directory:
  title: 'Gérer l''annuaire d''événements'
```

Dans le fichier de routing, on remplace `_permission: 'access content'` par `_permission: 'manage event directory'` : Drupal vérifie alors automatiquement que l'utilisateur courant possède cette permission (via son rôle) avant d'autoriser l'accès à la route, sans code de vérification manuel dans le contrôleur.
