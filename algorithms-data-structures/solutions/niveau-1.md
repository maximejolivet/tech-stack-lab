# Solutions — Niveau 1 (Bases)

## Exercice 1

- `a` : **O(1)** — accès direct au premier élément par index, indépendant de la taille de la liste.
- `b` : **O(n)** — un seul parcours de la liste, une opération par élément.
- `c` : **O(n²)** — une boucle imbriquée sur la même liste, donc `n × n` itérations.

## Exercice 2

```python
def parentheses_valides(s: str) -> bool:
    paires = {')': '(', ']': '[', '}': '{'}
    pile = []
    for char in s:
        if char in '([{':
            pile.append(char)
        elif char in ')]}':
            if not pile or pile.pop() != paires[char]:
                return False
    return len(pile) == 0
```

## Exercice 3

```python
from collections import deque

file = deque(["Alice", "Bob", "Charlie", "Diana"])
print(file.popleft())  # Alice
print(file.popleft())  # Bob
print(file)             # deque(['Charlie', 'Diana'])
```

## Exercice 4

```python
def compter_occurrences(mots: list[str]) -> dict[str, int]:
    compteur = {}
    for mot in mots:
        compteur[mot] = compteur.get(mot, 0) + 1
    return compteur
```

## Exercice 5

```python
def recherche_binaire(liste_triee: list[int], cible: int) -> int:
    gauche, droite = 0, len(liste_triee) - 1
    while gauche <= droite:
        milieu = (gauche + droite) // 2
        if liste_triee[milieu] == cible:
            return milieu
        elif liste_triee[milieu] < cible:
            gauche = milieu + 1
        else:
            droite = milieu - 1
    return -1

recherche_binaire([1, 3, 5, 7, 9, 11], 7)  # 3
recherche_binaire([1, 3, 5, 7, 9, 11], 4)  # -1
```
