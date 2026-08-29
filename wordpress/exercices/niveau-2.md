# Exercices WordPress — Niveau 2 (Intermédiaire)

## Exercice 1 — Custom Post Type

Déclare un Custom Post Type `event` (label "Événements"), public, supportant `title` et `editor`, exposé dans l'API REST.

## Exercice 2 — WP_Query avec meta_query

Étant donné que chaque `event` possède un champ personnalisé `event_date` (format `YYYY-MM-DD`), écris une `WP_Query` qui récupère les 10 prochains événements dont `event_date` est supérieure ou égale à aujourd'hui, triés par `event_date` croissant.

## Exercice 3 — Shortcode

Crée un shortcode `[latest_events count="5"]` qui affiche une liste `<ul>` des N derniers événements (titre + date), avec l'attribut `count` par défaut à 5 s'il n'est pas fourni.

## Exercice 4 — Formulaire admin sécurisé par nonce

Écris un formulaire (dans une meta box) avec un champ texte `event_location`, protégé par `wp_nonce_field`, et la fonction de sauvegarde associée sur le hook `save_post` qui vérifie le nonce avant d'appeler `update_post_meta`.

## Exercice 5 — pre_get_posts

La page d'archive du CPT `event` doit toujours trier les événements par `event_date` croissante (au lieu du tri par date de publication par défaut), sans créer de `WP_Query` secondaire. Utilise le hook approprié.
