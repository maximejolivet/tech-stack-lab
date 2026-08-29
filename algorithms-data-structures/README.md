# Algorithmes & Structures de données

## 1. Introduction

Les algorithmes et structures de données sont les bases théoriques indépendantes de tout langage ou framework : comment organiser des données en mémoire et comment raisonner sur le coût (temps, mémoire) d'un traitement à mesure que le volume de données grandit. Ce dossier utilise **Python** pour tous les exemples de code (syntaxe concise, proche du pseudocode) — voir [`../python/`](../python/) pour les bases du langage ; les concepts restent transposables à n'importe quel langage, y compris [`../javascript/`](../javascript/).

**À quoi ça sert ?**
- Choisir la bonne structure de données pour un problème donné (ex. recherche fréquente → hash map, ordre + insertions fréquentes → arbre équilibré).
- Repérer et corriger un traitement qui dégrade mal avec le volume de données (boucle imbriquée sur une grosse collection, recherche linéaire répétée).
- Réussir les entretiens techniques, qui s'appuient massivement sur ce socle.

**Où ça se situe ?** C'est un socle théorique transverse, mobilisé implicitement dans absolument tout code manipulant des collections de données — pas une techno ou un framework à installer.

**Enjeux**
- Un mauvais choix de structure ou d'algorithme peut transformer un traitement de O(n) en O(n²), invisible sur un jeu de données de test mais catastrophique en production à l'échelle.
- Savoir nommer et estimer la complexité d'un traitement permet de discuter architecture (voir [`../system-design/`](../system-design/)) avec du vocabulaire précis plutôt que des intuitions vagues.

**Pièges**
- Optimiser prématurément un code qui n'est jamais un goulot d'étranglement réel — la plupart des applications web passent leur temps en I/O (réseau, base de données), pas en calcul algorithmique pur.
- Confondre complexité théorique et performance réelle : un algorithme en O(n log n) avec une grosse constante peut être plus lent qu'un O(n²) naïf sur de petits volumes.

## 2. Prérequis

- Bases de programmation : variables, boucles, conditions, fonctions, récursivité (voir [`../python/`](../python/) ou [`../javascript/`](../javascript/)).
- Aucune notion mathématique avancée requise pour les bases ; l'analyse de complexité (section 3) demande simplement de la logique.

## 3. Rappel des bases 🟢

### 01 - Notation Big-O

**Explication** — La notation Big-O décrit comment le temps d'exécution (ou la mémoire) d'un algorithme croît en fonction de la taille de l'entrée `n`, en ignorant les constantes et les termes non dominants. Elle répond à la question "que se passe-t-il quand `n` devient très grand ?".

```python
def contient(liste, valeur):        # O(n) : dans le pire cas, parcourt tout
    for item in liste:
        if item == valeur:
            return True
    return False

def premier_element(liste):          # O(1) : temps constant, indépendant de n
    return liste[0]
```

**Ordres courants, du meilleur au pire** : O(1) constant, O(log n) logarithmique, O(n) linéaire, O(n log n) quasi-linéaire, O(n²) quadratique, O(2ⁿ) exponentiel.

**Bonne pratique** : avant d'optimiser, identifier la complexité actuelle et vérifier qu'elle est réellement un problème sur les volumes de données concernés — pas d'optimisation sans mesure.

### 02 - Tableaux (arrays / listes)

**Explication** — Structure contiguë en mémoire, accès par index en O(1), mais insertion/suppression en milieu de liste en O(n) (décalage des éléments suivants).

```python
fruits = ["pomme", "banane", "cerise"]
fruits[1]                # O(1) — accès direct par index
fruits.append("kiwi")     # O(1) amorti — ajout en fin
fruits.insert(0, "abricot")  # O(n) — décale tous les éléments
```

**Bonne pratique** : privilégier les ajouts/suppressions en fin de tableau (O(1)) plutôt qu'en début ou au milieu (O(n)) quand l'ordre le permet.

### 03 - Listes chaînées (linked lists)

**Explication** — Chaque élément (nœud) contient une valeur et une référence vers le nœud suivant. Insertion/suppression en tête en O(1) (pas de décalage), mais accès par index en O(n) (il faut parcourir depuis le début).

```python
class Noeud:
    def __init__(self, valeur, suivant=None):
        self.valeur = valeur
        self.suivant = suivant

tete = Noeud(1, Noeud(2, Noeud(3)))  # liste chaînée 1 -> 2 -> 3
```

**Cas d'usage** : utile quand les insertions/suppressions fréquentes en tête priment sur l'accès aléatoire — en pratique, rarement utilisée directement en Python (les listes natives suffisent), mais le concept sous-tend les files/piles et certaines structures avancées (arbres, graphes).

### 04 - Piles (stacks) et files (queues)

**Explication** — Une **pile** suit l'ordre LIFO (Last In, First Out — dernier entré, premier sorti) : empiler/dépiler en O(1). Une **file** suit l'ordre FIFO (First In, First Out) : enfiler/défiler en O(1) avec la bonne structure.

```python
pile = []
pile.append("a"); pile.append("b")
pile.pop()  # "b" — dernier entré, premier sorti

from collections import deque
file = deque()
file.append("a"); file.append("b")
file.popleft()  # "a" — premier entré, premier sorti
```

**Erreur fréquente** : utiliser une liste Python (`list.pop(0)`) comme file — `pop(0)` est en O(n) (décale tous les éléments restants). Utiliser `collections.deque`, conçu pour des opérations O(1) aux deux extrémités.

**Cas d'usage** : pile → historique de navigation, annulation (undo), évaluation d'expressions ; file → traitement de tâches dans l'ordre d'arrivée, parcours en largeur (BFS, voir section 4).

### 05 - Hash maps (dictionnaires)

**Explication** — Structure clé-valeur avec accès, insertion et suppression en O(1) en moyenne, grâce à une fonction de hachage qui calcule directement l'emplacement en mémoire d'une clé.

```python
ages = {"Alice": 30, "Bob": 25}
ages["Charlie"] = 35   # O(1) en moyenne
"Alice" in ages          # O(1) en moyenne — bien plus rapide qu'un parcours de liste
```

**Erreur fréquente** : chercher une valeur dans une liste avec `in` de façon répétée dans une boucle (O(n) à chaque appel, donc O(n²) au total) alors qu'un dictionnaire ou un `set` ramènerait chaque recherche à O(1).

**Bonne pratique** : dès qu'un problème implique "vérifier si j'ai déjà vu cet élément" ou "compter des occurrences", penser hash map/set avant de penser boucle imbriquée.

### 06 - Recherche binaire (binary search)

**Explication** — Sur une collection **triée**, recherche en O(log n) en divisant l'espace de recherche par deux à chaque étape, au lieu de parcourir séquentiellement (O(n)).

```python
def recherche_binaire(liste_triee, cible):
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
```

**Erreur fréquente** : appliquer une recherche binaire sur une collection non triée — le résultat est incorrect silencieusement, pas une erreur explicite.

### 07 - Tri : merge sort et quicksort

**Explication** — Deux algorithmes de tri en O(n log n) en moyenne, bien plus efficaces que les tris naïfs en O(n²) (tri à bulles, tri par insertion) sur de grandes collections. **Merge sort** divise puis fusionne en triant ; **quicksort** partitionne autour d'un pivot.

```python
def merge_sort(liste):
    if len(liste) <= 1:
        return liste
    milieu = len(liste) // 2
    gauche = merge_sort(liste[:milieu])
    droite = merge_sort(liste[milieu:])
    return fusionner(gauche, droite)

def fusionner(a, b):
    resultat = []
    i = j = 0
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            resultat.append(a[i]); i += 1
        else:
            resultat.append(b[j]); j += 1
    return resultat + a[i:] + b[j:]
```

**Bonne pratique** : en pratique, utiliser le tri natif du langage (`sorted()` en Python, en O(n log n) et fortement optimisé) plutôt que réimplémenter un tri — l'intérêt de merge sort/quicksort ici est de comprendre le principe "diviser pour régner", réutilisé dans beaucoup d'autres algorithmes.

### 08 - Récursivité

**Explication** — Une fonction qui s'appelle elle-même, avec un **cas de base** (condition d'arrêt) et un **cas récursif** qui rapproche du cas de base. Base de nombreux algorithmes (parcours d'arbres, diviser pour régner).

```python
def factorielle(n):
    if n <= 1:            # cas de base
        return 1
    return n * factorielle(n - 1)   # cas récursif
```

**Erreur fréquente** : oublier ou mal poser le cas de base → récursion infinie jusqu'au dépassement de la pile d'appels (`RecursionError` en Python).

**Bonne pratique** : pour des données volumineuses ou une récursion profonde, envisager une version itérative — Python ne fait pas d'optimisation de la récursion terminale (tail-call optimization) contrairement à certains langages.

## 4. Concepts intermédiaires 🟡

- **Arbres binaires de recherche (BST)** : chaque nœud a au plus deux enfants, avec la propriété "gauche < nœud < droite" — recherche, insertion, suppression en O(log n) **si l'arbre est équilibré**, dégradé en O(n) si déséquilibré (cas pathologique : insertion d'éléments déjà triés, qui produit une liste chaînée déguisée).

```python
class NoeudBST:
    def __init__(self, valeur):
        self.valeur = valeur
        self.gauche = None
        self.droite = None

def inserer(noeud, valeur):
    if noeud is None:
        return NoeudBST(valeur)
    if valeur < noeud.valeur:
        noeud.gauche = inserer(noeud.gauche, valeur)
    else:
        noeud.droite = inserer(noeud.droite, valeur)
    return noeud
```

- **Arbres équilibrés (aperçu)** : AVL, arbres rouge-noir — maintiennent automatiquement l'équilibre à chaque insertion/suppression pour garantir O(log n) dans le pire cas. Rarement réimplémentés à la main en pratique (les bibliothèques standard ou bases de données les fournissent), mais le concept explique pourquoi un index de base de données (voir [`../mysql/`](../mysql/)) reste performant même après de nombreuses écritures.
- **Graphes — parcours en largeur (BFS) et en profondeur (DFS)** : un graphe est un ensemble de nœuds reliés par des arêtes. BFS explore niveau par niveau (via une file) — utile pour trouver le plus court chemin en nombre d'arêtes. DFS explore une branche jusqu'au bout avant de revenir en arrière (via une pile ou la récursion) — utile pour détecter des cycles ou explorer toutes les possibilités.

```python
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
```

- **Plus court chemin (aperçu)** : Dijkstra pour un graphe à poids positifs — utilise une file de priorité pour toujours explorer le nœud le plus proche non encore visité en premier. Sous-jacent à des cas d'usage concrets (calcul d'itinéraire, routage réseau).
- **Programmation dynamique (introduction)** : technique qui décompose un problème en sous-problèmes qui se chevauchent, et **mémorise** (cache) leurs résultats pour éviter de les recalculer — transforme souvent une complexité exponentielle en polynomiale.

```python
def fibonacci_memo(n, cache={}):
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fibonacci_memo(n - 1, cache) + fibonacci_memo(n - 2, cache)
    return cache[n]
```

- **Quand s'en soucier réellement en développement web** : la grande majorité du code applicatif quotidien n'a pas besoin de réimplémenter ces algorithmes — l'enjeu est de **reconnaître** les patterns (recherche répétée → hash map, hiérarchie → arbre, dépendances → graphe) et d'utiliser les structures/fonctions déjà fournies par le langage ou une bibliothèque, plutôt que d'écrire une boucle imbriquée naïve par réflexe.

## 5. Concepts avancés 🟠🔴

- **Complexité en espace (space complexity)** : au-delà du temps, mesurer la mémoire additionnelle utilisée par un algorithme (ex. la mémoïsation de la section 4 échange de la mémoire contre du temps) — un compromis fréquent à expliciter.
- **Tables de hachage — collisions et redimensionnement** : comprendre que le O(1) moyen d'un hash map suppose une bonne fonction de hachage et une gestion des collisions (chaînage ou adressage ouvert) ; dans le pire cas (beaucoup de collisions), les opérations dégradent vers O(n).
- **Tri : stabilité et cas limites** : un tri "stable" préserve l'ordre relatif des éléments égaux (important quand on trie par plusieurs critères successifs) — merge sort est stable, quicksort ne l'est pas nativement.
- **Structures avancées (aperçu)** : tas binaire (heap, base des files de priorité, O(log n) pour insérer/extraire le minimum/maximum), trie (arbre préfixe, utile pour l'autocomplétion), union-find (détection de composantes connexes, utile pour Kruskal ou la détection de cycles).
- **Limites du calcul de complexité en entretien vs en production** : un exercice d'entretien isole l'algorithme ; en production, la latence réseau, les I/O disque et les temps d'accès base de données dominent très largement le temps de calcul pur — la vraie compétence utile au quotidien est de savoir repérer les cas où la complexité algorithmique *devient* le facteur limitant (traitement en mémoire sur de gros volumes, boucle sur une collection qui grossit avec le nombre d'utilisateurs).
- **P vs NP (aperçu culturel)** : certains problèmes (ex. voyageur de commerce) n'ont pas d'algorithme connu en temps polynomial pour trouver la solution optimale — savoir reconnaître qu'on est face à ce type de problème évite de chercher une solution exacte parfaite là où une heuristique ou une approximation suffit en pratique.

## 6. Commandes / syntaxe à connaître

```python
sorted(liste)                       # tri natif, O(n log n)
sorted(liste, key=lambda x: x.age)   # tri par critère personnalisé
liste.append(x); liste.pop()         # pile — O(1) en fin de liste
from collections import deque
deque().append(x); deque().popleft()  # file — O(1) aux deux extrémités
{cle: valeur for cle, valeur in ...}  # dict comprehension
set(liste)                            # déduplication, tests d'appartenance O(1)
from functools import lru_cache
@lru_cache(maxsize=None)              # mémoïsation automatique d'une fonction récursive
def f(n): ...
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Analyseur de logs avec structures adaptées**

Construire un script Python qui traite un fichier de logs (une ligne = un événement horodaté avec une adresse IP et une URL visitée), en choisissant à chaque étape la structure de données adaptée :
- Compter le nombre de visites par IP (hash map) et afficher le top 5 (tri par valeur).
- Détecter les IP suspectes (plus de N requêtes en moins d'une seconde) avec une file glissante (`deque`).
- Construire un graphe simple des transitions entre pages (IP visite `/a` puis `/b` → arête `/a → /b`) et faire un BFS pour trouver le chemin de navigation le plus court entre deux pages données.
- Mesurer et comparer le temps d'exécution d'une recherche linéaire vs un hash map pour vérifier si une IP est dans une liste noire de 100 000 entrées.
- Bonus : utiliser la mémoïsation pour éviter de recalculer les statistiques déjà vues sur un fichier de logs qui grossit en continu.

## Checklist

- [ ] Comprendre les fondamentaux (Big-O, tableaux, listes chaînées, piles/files, hash maps)
- [ ] Savoir écrire une recherche binaire et un tri par fusion sans aide
- [ ] Maîtriser la syntaxe principale (structures natives Python : list, dict, set, deque)
- [ ] Comprendre les concepts importants (arbres, graphes, BFS/DFS, programmation dynamique)
- [ ] Savoir estimer la complexité temporelle d'un bout de code donné
- [ ] Connaître les bonnes pratiques (choisir la bonne structure, éviter la boucle imbriquée par réflexe)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (tas, trie, union-find, limites pratiques du calcul de complexité)

## 10. Ressources

- [roadmap.sh — Computer Science](https://roadmap.sh/computer-science) — vue d'ensemble incluant algorithmes et structures de données.
- [roadmap.sh — Data Structures & Algorithms](https://roadmap.sh/datastructures-and-algorithms) — roadmap dédiée si disponible ; à défaut, la roadmap Computer Science ci-dessus couvre le même socle.
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/) — référence rapide des complexités des structures et algorithmes courants.
- [VisuAlgo](https://visualgo.net/) — visualisations interactives d'algorithmes et structures de données.
- [`../python/`](../python/) — bases du langage utilisé dans ce dossier.
