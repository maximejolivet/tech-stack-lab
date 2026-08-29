# Exercices Algorithmes & Structures de données — Niveau 1 (Bases)

## Exercice 1 — Complexité de trois fonctions

Pour chacune de ces trois fonctions Python, donne sa complexité en notation Big-O et justifie en une phrase :

```python
def a(liste):
    return liste[0] if liste else None

def b(liste):
    total = 0
    for x in liste:
        total += x
    return total

def c(liste):
    paires = []
    for x in liste:
        for y in liste:
            paires.append((x, y))
    return paires
```

## Exercice 2 — Pile pour vérifier des parenthèses

Écris une fonction `parentheses_valides(s: str) -> bool` qui vérifie qu'une chaîne contenant `(`, `)`, `[`, `]`, `{`, `}` a ses parenthèses/crochets/accolades correctement ouverts et fermés, à l'aide d'une pile.

## Exercice 3 — File avec deque

Simule une file d'attente de clients avec `collections.deque` : ajoute 4 noms de clients, sers les deux premiers (affiche leur nom en les retirant), puis affiche l'état final de la file.

## Exercice 4 — Compter les occurrences avec un hash map

Étant donné une liste de mots, écris une fonction `compter_occurrences(mots: list[str]) -> dict[str, int]` qui retourne un dictionnaire associant chaque mot à son nombre d'occurrences, en un seul parcours de la liste.

## Exercice 5 — Recherche binaire

Implémente `recherche_binaire(liste_triee: list[int], cible: int) -> int` qui retourne l'index de `cible` dans `liste_triee`, ou `-1` si absent. Teste-la sur `[1, 3, 5, 7, 9, 11]` avec `cible = 7` puis `cible = 4`.
