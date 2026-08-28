# Niveau 3 — Avancé

## Exercice 3.1 — Asyncio

Simule le traitement de 3 "requêtes réseau" (coroutines qui font `await asyncio.sleep(...)` avec des délais différents puis retournent un résultat) en parallèle avec `asyncio.gather`. Affiche le temps total d'exécution et vérifie qu'il est proche du délai le plus long, pas de la somme des délais.

## Exercice 3.2 — GIL et multiprocessing

Écris une fonction CPU-bound (ex. calculer la somme des carrés de 10 millions de nombres). Chronomètre son exécution séquentielle sur 4 appels, puis avec `multiprocessing.Pool(4)`. Explique en 3-4 lignes pourquoi un équivalent avec `threading.Thread` n'aurait pas accéléré le calcul, à cause du GIL.

## Exercice 3.3 — Context manager custom

Écris un context manager custom (via une classe avec `__enter__`/`__exit__`, ou via `@contextmanager`) qui chronomètre un bloc de code et affiche sa durée à la sortie, même si une exception est levée à l'intérieur du bloc.

## Exercice 3.4 — Typage statique avec mypy

Ajoute des type hints complets (paramètres et retours) à une fonction `def process(items, transform):` qui applique `transform` à chaque élément de `items` et retourne la liste résultat (utilise `Callable` et `list` génériques). Vérifie qu'aucune erreur ne remonte avec `mypy`, puis introduis volontairement une incohérence de type et observe l'erreur signalée.
