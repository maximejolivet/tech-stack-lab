# Exercices Testing — Niveau 3 (Avancé)

## Exercice 1 — Mutation testing

Un outil de mutation testing remplace `if (age >= 18)` par `if (age > 18)` dans le code source, relance la suite de tests, et constate qu'aucun test n'échoue. Explique ce que révèle ce "mutant survivant", et quel test manquant il faudrait ajouter.

## Exercice 2 — Contract testing

Un service `orders-api` consomme l'API d'un service `users-api`. Explique en 3-4 phrases pourquoi un test de contrat (Pact) entre les deux équipes est préférable à un test d'intégration bout-en-bout qui ferait tourner les deux vrais services à chaque run de CI, et ce que chaque côté vérifie concrètement.

## Exercice 3 — Property-based testing

Pour une fonction `reverse(list)` qui inverse une liste, propose deux propriétés générales qu'un outil de property-based testing pourrait vérifier automatiquement sur des centaines de listes générées (au lieu d'exemples fixes écrits à la main).

## Exercice 4 — Test de charge

Explique en 2-3 phrases la différence entre un test fonctionnel qui vérifie qu'un endpoint retourne la bonne réponse, et un test de charge qui vérifie que ce même endpoint tient une charge de 1000 requêtes/seconde — et pourquoi ils ont des critères de succès fondamentalement différents.

## Exercice 5 — Culture de test

Une équipe a un test qui échoue de façon intermittente depuis 3 semaines ; personne ne l'a corrigé, et plusieurs développeurs ont pris l'habitude de relancer la pipeline jusqu'à ce qu'il passe. Explique en 3-4 phrases le risque à moyen terme de cette pratique sur la confiance de l'équipe dans l'ensemble de la suite de tests, au-delà de ce seul test.
