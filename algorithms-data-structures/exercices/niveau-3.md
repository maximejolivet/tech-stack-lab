# Exercices Algorithmes & Structures de données — Niveau 3 (Avancé)

## Exercice 1 — Plus court chemin non pondéré

En réutilisant un BFS, écris une fonction `plus_court_chemin(graphe, depart, arrivee) -> list` qui retourne la liste des nœuds du plus court chemin (en nombre d'arêtes) entre `depart` et `arrivee`, ou `None` si aucun chemin n'existe.

## Exercice 2 — Programmation dynamique : somme de sous-tableau maximale

Implémente l'algorithme de Kadane pour trouver, en un seul parcours O(n), la somme maximale d'un sous-tableau contigu dans une liste d'entiers pouvant contenir des négatifs (ex. `[-2, 1, -3, 4, -1, 2, 1, -5, 4]` → `6`).

## Exercice 3 — Tas binaire (heap) : top-K

En utilisant le module `heapq` de Python, écris une fonction `top_k(liste: list[int], k: int) -> list[int]` qui retourne les k plus grandes valeurs d'une liste, plus efficacement qu'un tri complet suivi d'un slicing sur de très grandes listes. Explique en une phrase pourquoi.

## Exercice 4 — Union-Find (détection de cycle)

Implémente une structure Union-Find minimale (`find`, `union`) et utilise-la pour détecter si l'ajout d'une arête à un graphe non orienté créerait un cycle, étant donné une liste d'arêtes déjà ajoutées.

## Exercice 5 — Analyse de complexité en espace

Pour la version mémoïsée de `fibonacci` de l'exercice 5 du niveau 2, donne sa complexité en **temps** et en **espace** (mémoire utilisée par le cache), et explique en 2-3 phrases le compromis effectué par rapport à la version itérative qui ne garde que les deux derniers termes.
