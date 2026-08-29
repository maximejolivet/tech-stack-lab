# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```python
def merge_sort(liste):
    if len(liste) <= 1:
        return liste
    milieu = len(liste) // 2
    gauche = merge_sort(liste[:milieu])
    droite = merge_sort(liste[milieu:])
    return fusionner(gauche, droite)

def fusionner(a, b):
    resultat, i, j = [], 0, 0
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            resultat.append(a[i]); i += 1
        else:
            resultat.append(b[j]); j += 1
    return resultat + a[i:] + b[j:]

merge_sort([5, 2, 9, 1, 5, 6])  # [1, 2, 5, 5, 6, 9]
```

## Exercice 2

```python
def contient(noeud, valeur) -> bool:
    if noeud is None:
        return False
    if noeud.valeur == valeur:
        return True
    if valeur < noeud.valeur:
        return contient(noeud.gauche, valeur)
    return contient(noeud.droite, valeur)
```

## Exercice 3

```python
from collections import deque

def bfs(graphe, depart):
    visites = {depart}
    file = deque([depart])
    ordre = []
    while file:
        noeud = file.popleft()
        ordre.append(noeud)
        for voisin in graphe[noeud]:
            if voisin not in visites:
                visites.add(voisin)
                file.append(voisin)
    return ordre

graphe = {"A": ["B", "C"], "B": ["D"], "C": ["D"], "D": []}
bfs(graphe, "A")  # ['A', 'B', 'C', 'D']
```

## Exercice 4

```python
def a_des_doublons(liste: list) -> bool:
    vus = set()
    for x in liste:
        if x in vus:
            return True
        vus.add(x)
    return False
```

## Exercice 5

```python
def fibonacci_naif(n):
    if n <= 1:
        return n
    return fibonacci_naif(n - 1) + fibonacci_naif(n - 2)

from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci_memo(n):
    if n <= 1:
        return n
    return fibonacci_memo(n - 1) + fibonacci_memo(n - 2)
```

La version naïve recalcule les mêmes sous-problèmes un nombre exponentiel de fois (`fibonacci_naif(n-2)` est recalculé séparément par l'appel à `fibonacci_naif(n-1)` et par l'appel direct), ce qui donne une complexité en O(2ⁿ). La version mémoïsée ne calcule chaque valeur `fibonacci(k)` qu'une seule fois grâce au cache, ramenant la complexité à O(n) — à partir de `n = 35`, l'écart entre 2ⁿ et n devient assez grand pour que la version naïve prenne plusieurs secondes voire minutes, contre un temps imperceptible pour la version mémoïsée.
