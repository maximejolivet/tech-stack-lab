# Exercices WordPress — Niveau 3 (Avancé)

## Exercice 1 — Requête $wpdb sécurisée

Écris une requête `$wpdb` qui compte le nombre d'événements (`event`) créés par un auteur donné (`$userId`), en utilisant `prepare()` pour éviter toute injection SQL.

## Exercice 2 — Transient de cache

Le shortcode `[latest_events]` de l'exercice précédent (niveau 2) est coûteux à calculer. Modifie-le pour mettre son résultat HTML en cache dans un transient pendant 1 heure, avec une clé de cache qui dépend de l'attribut `count`.

## Exercice 3 — Invalidation du cache

Le transient de l'exercice 2 doit être supprimé automatiquement dès qu'un événement est publié ou modifié. Accroche-toi au hook approprié pour appeler `delete_transient`.

## Exercice 4 — Endpoint REST personnalisé

Crée un endpoint REST `GET /wp-json/mon-plugin/v1/events` (`register_rest_route`) qui retourne en JSON les événements à venir (titre, date, lien), en réutilisant la logique de l'exercice 2 du niveau 2.

## Exercice 5 — WP-Cron

Programme une tâche WP-Cron quotidienne qui supprime automatiquement les événements dont `event_date` est passée depuis plus de 90 jours. Explique en une phrase pourquoi cette tâche ne s'exécutera pas de façon fiable sur un site à très faible trafic, et quelle configuration recommander en production.
