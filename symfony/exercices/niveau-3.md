# Exercices — Niveau 3 (Avancé)

## Exercice 1 — Voter d'autorisation

Une entité `Article` a un champ `author`. Écris un `ArticleVoter` avec deux attributs `EDIT` et `DELETE` : seul l'auteur peut éditer/supprimer, sauf si l'utilisateur a le rôle `ROLE_ADMIN` (qui peut tout). Utilise-le dans un contrôleur avec `denyAccessUnlessGranted`.

## Exercice 2 — Événement métier + Subscriber asynchrone

Crée un événement `OrderPlacedEvent` (transporte l'id de commande), dispatche-le après la création d'une commande, et écris un `EventSubscriberInterface` qui logue l'événement. Puis explique comment tu le rendrais asynchrone avec Messenger (message dédié + handler) plutôt qu'un simple Event Subscriber, et dans quel cas ce choix se justifie.

## Exercice 3 — Architecture hexagonale légère

Propose un découpage `src/Domain/`, `src/Application/`, `src/Infrastructure/` pour un module "gestion de panier e-commerce" : quelles classes vivent où, et pourquoi le `Domain` ne doit dépendre d'aucune classe Doctrine/Symfony. Donne un exemple de signature d'interface définie dans `Domain` et implémentée dans `Infrastructure`.

## Exercice 4 — Performance en production

Liste au moins 4 réglages ou pratiques à vérifier avant une mise en production Symfony (env, opcache, cache warmup, secrets) et explique l'impact de chacun s'il est oublié.
