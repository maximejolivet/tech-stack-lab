# Niveau 2 — Intermédiaire

## Exercice 2.1 — Mode worker minimal

Écris un script worker minimal (sans framework) qui incrémente un compteur en mémoire à chaque requête et renvoie sa valeur. Constate que le compteur persiste entre les requêtes — ce qui ne serait pas le cas en mode classic ou en PHP-FPM.

## Exercice 2.2 — Piège de l'état partagé

Modifie le script précédent pour simuler un bug courant : une variable globale contenant "le dernier utilisateur connecté" qui n'est jamais réinitialisée entre deux requêtes de deux utilisateurs différents. Envoie deux requêtes successives simulant deux utilisateurs et observe la fuite d'état. Corrige le bug.

## Exercice 2.3 — Intégration framework (recherche)

Sans forcément l'exécuter, documente en quelques lignes comment Symfony Runtime ou Laravel Octane évitent le problème de l'exercice 2.2 par défaut pour le conteneur de services applicatif.
