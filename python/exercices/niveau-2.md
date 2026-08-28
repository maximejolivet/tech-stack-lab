# Niveau 2 — Intermédiaire

## Exercice 2.1 — POO et dunder methods

Crée une classe `Rectangle` (`width`, `height`) avec une méthode `area()`. Implémente `__str__` (affichage lisible) et `__eq__` (deux rectangles sont égaux si mêmes dimensions). Vérifie avec `print()` et `==`.

## Exercice 2.2 — Compréhensions

À partir d'une liste de noms (`list[str]`), produis (avec des compréhensions, sans boucle `for` manuelle) :
1. La liste des noms en majuscules.
2. La liste des noms de plus de 4 lettres.
3. Un dict `{nom: longueur}` pour chaque nom.

## Exercice 2.3 — Décorateur de chronométrage

Écris un décorateur `@timed` qui affiche le temps d'exécution (en secondes) de la fonction décorée, en utilisant `time.perf_counter()`. Applique-le à une fonction qui fait une boucle longue.

## Exercice 2.4 — Générateur

Écris un générateur `even_numbers(limit)` qui produit (`yield`) les nombres pairs de 0 à `limit` sans construire de liste intermédiaire. Consomme-le dans une boucle `for`.

## Exercice 2.5 — Exception custom

Écris une fonction `withdraw(balance, amount)` qui lève une exception custom `InsufficientFundsError` (héritant d'`Exception`) si `amount > balance`. Catche-la proprement dans un `try/except` et affiche un message clair.
