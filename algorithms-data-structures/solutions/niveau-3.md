# Solutions — Niveau 3 (Avancé)

## Exercice 1

```python
from collections import deque

def plus_court_chemin(graphe, depart, arrivee):
    if depart == arrivee:
        return [depart]
    visites = {depart}
    file = deque([[depart]])
    while file:
        chemin = file.popleft()
        dernier = chemin[-1]
        for voisin in graphe[dernier]:
            if voisin == arrivee:
                return chemin + [voisin]
            if voisin not in visites:
                visites.add(voisin)
                file.append(chemin + [voisin])
    return None
```

## Exercice 2

```python
def max_sous_tableau(liste: list[int]) -> int:
    max_courant = max_global = liste[0]
    for x in liste[1:]:
        max_courant = max(x, max_courant + x)
        max_global = max(max_global, max_courant)
    return max_global

max_sous_tableau([-2, 1, -3, 4, -1, 2, 1, -5, 4])  # 6
```

## Exercice 3

```python
import heapq

def top_k(liste: list[int], k: int) -> list[int]:
    return heapq.nlargest(k, liste)
```

`heapq.nlargest` maintient un tas de taille `k` en O(n log k), au lieu de trier toute la liste en O(n log n) puis de découper les k premiers — plus efficace quand `k` est petit devant `n` (on n'a jamais besoin d'ordonner totalement les éléments qui ne font pas partie du top-k).

## Exercice 4

```python
class UnionFind:
    def __init__(self):
        self.parent = {}

    def find(self, x):
        self.parent.setdefault(x, x)
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # compression de chemin
        return self.parent[x]

    def union(self, a, b):
        racine_a, racine_b = self.find(a), self.find(b)
        if racine_a == racine_b:
            return False  # même composante : ajouter cette arête créerait un cycle
        self.parent[racine_a] = racine_b
        return True

uf = UnionFind()
for a, b in [("A", "B"), ("B", "C")]:
    uf.union(a, b)

uf.union("A", "C")  # False -> créerait un cycle
```

## Exercice 5

La version mémoïsée est en **O(n) en temps** (chaque valeur de 0 à n calculée une seule fois) et **O(n) en espace** (le cache stocke une entrée par valeur de `n` déjà calculée, plus la pile d'appels récursifs jusqu'à profondeur n). Une version itérative qui ne garde que les deux derniers termes reste en O(n) en temps mais descend à **O(1) en espace** : elle sacrifie la possibilité de relire instantanément n'importe quel `fibonacci(k)` déjà calculé (utile si la fonction est rappelée plusieurs fois avec des `n` différents) au profit d'une empreinte mémoire constante — un compromis à faire selon si le résultat est consommé une seule fois ou réutilisé.
