# Solutions — Niveau 1 (Bases)

## Exercice 1

```yaml
name: 'Annuaire Événements'
type: module
description: 'Annuaire des événements.'
core_version_requirement: ^10
package: Custom
```

## Exercice 2

```php
$node = \Drupal\node\Entity\Node::load(42);
$node->setTitle('Titre mis à jour');
$node->save();
```

## Exercice 3

```php
function mon_module_node_presave(\Drupal\node\NodeInterface $node) {
    if ($node->bundle() === 'article' && $node->get('field_summary')->isEmpty()) {
        $body = $node->get('body')->value ?? '';
        $node->set('field_summary', mb_substr(strip_tags($body), 0, 100));
    }
}
```

## Exercice 4

```twig
{# node--event.html.twig #}
<h1>{{ label }}</h1>
<p class="location">{{ content.field_location }}</p>
```

## Exercice 5

`drush config:export` exporte toute la configuration active du site (types de contenu, champs, vues, permissions...) vers des fichiers YAML dans `config/sync/`. Versionner ce dossier dans Git permet de déployer une évolution de configuration de façon reproductible entre environnements (dev → staging → prod), au lieu de reconfigurer manuellement l'admin sur chaque environnement. Si la configuration est modifiée directement en production sans réimport en local, les deux environnements divergent : le prochain `config:export` local écrasera silencieusement les changements faits en prod lors du déploiement suivant.
