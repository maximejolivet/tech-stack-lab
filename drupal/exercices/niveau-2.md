# Exercices Drupal — Niveau 2 (Intermédiaire)

## Exercice 1 — Plugin Block

Écris la classe PHP d'un Plugin Block `NextEventsBlock` (annotation `@Block`) qui affiche un simple message statique "Prochains événements" via la méthode `build()`.

## Exercice 2 — Route et contrôleur

Déclare une route `mon_module.hello` dans `mon_module.routing.yml` pour le chemin `/annuaire/bonjour`, avec la permission `access content`, pointant vers un contrôleur `HelloController::hello`. Écris ensuite ce contrôleur, qui retourne un tableau de rendu `['#markup' => 'Bonjour !']`.

## Exercice 3 — Service et injection de dépendances

Déclare un service `mon_module.event_helper` dans `mon_module.services.yml`, pointant vers une classe `EventHelper`. Écris ensuite le contrôleur `HelloController` modifié pour injecter ce service via son constructeur plutôt que d'utiliser `\Drupal::service()` directement.

## Exercice 4 — Requête via l'API Entité

Écris le code qui récupère, via `\Drupal::entityTypeManager()->getStorage('node')`, les IDs de tous les nodes publiés de type `event`, triés par `field_date` croissant.

## Exercice 5 — Permission personnalisée

Déclare une permission personnalisée `manage event directory` dans `mon_module.permissions.yml`, puis explique en une phrase comment la vérifier dans un contrôleur via `_permission` dans le fichier de routing.
