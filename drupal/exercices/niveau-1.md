# Exercices Drupal — Niveau 1 (Bases)

## Exercice 1 — Fichier .info.yml

Écris le fichier `mon_module.info.yml` minimal pour un module nommé "Annuaire Événements", compatible Drupal 10.

## Exercice 2 — Charger et modifier une entité

Écris le code qui charge le node d'ID 42, modifie son titre en "Titre mis à jour", et sauvegarde la modification.

## Exercice 3 — Hook de sauvegarde

Écris un `hook_node_presave` qui, pour tout node de type `article`, force le champ `field_summary` à contenir les 100 premiers caractères du corps (`body`) si `field_summary` est vide.

## Exercice 4 — Template Twig

Écris un template `node--event.html.twig` qui affiche le titre dans un `<h1>` et le champ `field_location` dans un `<p class="location">`.

## Exercice 5 — Export de configuration

Explique en 2-3 phrases ce que fait `drush config:export`, pourquoi versionner son résultat dans Git, et ce qui se passe si on modifie la configuration directement en production sans le réimporter en local ensuite.
