# Exercices Algorithmes & Structures de données — Niveau 2 (Intermédiaire)

## Exercice 1 — Merge sort

Implémente `merge_sort(liste: list[int]) -> list[int]` sans regarder la solution du README, puis vérifie-la sur `[5, 2, 9, 1, 5, 6]`.

## Exercice 2 — Arbre binaire de recherche : recherche

Étant donné la classe `NoeudBST` du README (section 4), écris une fonction `contient(noeud, valeur) -> bool` qui recherche récursivement une valeur dans un BST, en exploitant la propriété d'ordre (pas de parcours de tout l'arbre).

## Exercice 3 — Parcours en largeur (BFS)

Étant donné le graphe `{"A": ["B", "C"], "B": ["D"], "C": ["D"], "D": []}` (représenté par un dictionnaire d'adjacence), utilise un BFS pour afficher l'ordre de visite des nœuds en partant de `"A"`.

## Exercice 4 — Détection de doublons avec un set

Écris une fonction `a_des_doublons(liste: list) -> bool` en O(n) qui détecte si une liste contient des doublons, en utilisant un `set` plutôt qu'une double boucle.

## Exercice 5 — Fibonacci : naïf vs mémoïsé

Écris deux versions de `fibonacci(n)` : une récursive naïve (sans cache) et une mémoïsée (`@lru_cache` ou dictionnaire de cache). Explique en 2-3 phrases pourquoi la version naïve devient très lente à partir de `n = 35` environ, alors que la version mémoïsée reste rapide.
