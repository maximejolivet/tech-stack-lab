# Python

## 1. Introduction

Python est un langage interprété, à typage dynamique fort, orienté objet et multi-paradigme. Ce dossier couvre le **langage** — le framework [`../django/`](../django/) (l'équivalent Symfony/Laravel côté Python) a son propre dossier juste après.

**À quoi sert-il ?**
- Scripting et automatisation (le langage "glue" par excellence pour enchaîner des outils).
- Data science, machine learning et IA — écosystème dominant (NumPy, pandas, PyTorch, scikit-learn).
- Développement web backend (Django, Flask, FastAPI) et APIs.

**Où se situe-t-il dans une architecture web ?**
Côté serveur, comme PHP et Java : il reçoit une requête (via WSGI ou ASGI), exécute de la logique métier, retourne une réponse. Un serveur d'application Python (Gunicorn, Uvicorn) fait tourner des workers qui traitent les requêtes — proche du modèle PHP-FPM (process/worker par requête) plutôt que du modèle JVM/Node long-running à état partagé unique.

**Avantages**
- Syntaxe très lisible, indentation obligatoire qui force une mise en forme cohérente — courbe d'apprentissage douce.
- Écosystème de librairies immense, en particulier data/IA où c'est le standard de facto.
- Typage dynamique + REPL interactif : itération et prototypage très rapides comparé à un langage compilé comme Java.

**Limites**
- Performance brute inférieure à un langage compilé (Java) ou à un runtime optimisé JIT — le CPython "classique" reste un interpréteur.
- Le GIL (Global Interpreter Lock, voir section avancée) limite le parallélisme réel du multithreading sur du calcul CPU-bound.
- Typage dynamique par défaut : les erreurs de type ne sont détectées qu'à l'exécution, sauf ajout volontaire de type hints + outillage (mypy) — différence structurante avec Java.

## 2. Prérequis

- POO déjà pratiquée en PHP ([`../php/`](../php/)) ou Java ([`../java/`](../java/)) — les concepts (classes, héritage, interfaces) sont transférables.
- Typage dynamique déjà rencontré en JavaScript ([`../javascript/`](../javascript/)) — Python partage cette flexibilité, avec des règles de coercition plus strictes (pas d'équivalent du `==` laxiste de JS/PHP).
- Python installé en local (`python3 --version`), idéalement 3.11+ pour bénéficier des dernières fonctionnalités abordées ici.

## 3. Rappel des bases 🟢

### 01 - L'interpréteur CPython et l'exécution

**Explication** — CPython (l'implémentation de référence) lit le fichier `.py`, le compile en bytecode interne (`.pyc`, mis en cache), puis l'exécute via une boucle d'interprétation — contrairement à Java où le bytecode est un artefact durable et portable, ici c'est un détail d'implémentation invisible pour le développeur.

```bash
python3 script.py       # exécute directement, pas d'étape de build manuelle
python3 -i script.py    # exécute puis reste en mode interactif (REPL)
python3                 # REPL seul, pour tester rapidement une expression
```

**Cas d'usage** : le REPL est un outil de travail à part entière en Python — tester une expression, inspecter un objet (`dir(obj)`, `help(obj)`) avant de l'écrire dans un fichier.

**Bonne pratique** : ne jamais versionner le dossier `__pycache__/` (équivalent de `vendor/` ou `node_modules/`) — à ajouter systématiquement au `.gitignore`.

### 02 - Structure minimale et indentation significative

**Explication** — Il n'y a pas d'accolades : les blocs (corps de fonction, de boucle, de condition) sont délimités par l'indentation elle-même (4 espaces par convention, PEP 8). Une indentation incohérente est une erreur de syntaxe, pas un simple style.

```python
def greet(name):
    if name:
        print(f"Bonjour {name}")
    else:
        print("Bonjour inconnu")
```

**Erreur fréquente** : mélanger tabulations et espaces dans un même fichier — Python 3 lève une `TabError` explicite, mais l'erreur peut être déroutante si l'éditeur ne l'affiche pas clairement.

**Bonne pratique** : configurer l'éditeur pour insérer 4 espaces à la place des tabulations (comportement par défaut de la plupart des IDE modernes en `.py`).

### 03 - Types dynamiques de base

**Explication** — `int`, `float`, `str`, `bool`, `NoneType` (`None`, équivalent de `null`). Le typage est **dynamique** (une variable peut changer de type) et **fort** (pas de coercition implicite silencieuse entre types incompatibles, contrairement à PHP/JS).

```python
age = 30          # int
price = 19.99      # float
name = "Max"        # str
active = True         # bool
result = None           # absence de valeur

# "5" + 3  → TypeError, contrairement à PHP/JS qui coercent silencieusement
int("5") + 3           # 8, conversion explicite obligatoire
```

**Erreur fréquente** : s'attendre à une coercition automatique `str` + `int` comme en JS (`"5" + 3` → `"53"`) — Python lève une `TypeError` franche, il faut convertir explicitement.

### 04 - Variables et typage dynamique fort

**Explication** — Une variable est un simple nom lié à un objet ; elle peut être réassignée à un objet d'un type différent sans déclaration préalable. Depuis Python 3.5+, des **type hints** optionnels documentent l'intention sans être vérifiés à l'exécution (voir avancé pour `mypy`).

```python
x = 5
x = "maintenant une chaîne"  # valide : la variable n'est pas typée, l'objet qu'elle référence l'est

def add(a: int, b: int) -> int:   # type hints : indicatifs, non contraignants au runtime
    return a + b

add("2", "3")  # fonctionne quand même à l'exécution (concatène les strings) : les hints ne sont pas appliqués sans outillage
```

**Bonne pratique** : ajouter des type hints sur toute signature de fonction publique dès qu'un projet dépasse un script jetable — ça documente l'intention et permet à mypy/l'IDE de détecter des erreurs avant l'exécution.

### 05 - Structures de contrôle

**Explication** — `if/elif/else`, `for` (itère sur un itérable, pas sur un compteur comme le `for` C-style de PHP/Java), `while`. Depuis Python 3.10, `match` apporte le pattern matching structurel (proche du `match` de PHP 8 ou du `switch` moderne de Java).

```python
for i in range(5):        # équivalent d'un for(i=0; i<5; i++)
    print(i)

for item in ["a", "b", "c"]:  # itération directe sur les éléments
    print(item)

status = 404
match status:
    case 200:
        print("OK")
    case 404 | 410:
        print("Not Found")
    case _:
        print("Autre")
```

**Erreur fréquente** : chercher un `for (int i = 0; i < n; i++)` classique — Python n'a pas cette forme, `range(n)` est l'idiome équivalent.

### 06 - Structures de données : listes, tuples, dicts, sets

**Explication** — `list` (mutable, ordonnée, équivalent d'un tableau indexé PHP), `tuple` (immuable, ordonnée), `dict` (clé-valeur, équivalent d'un tableau associatif PHP ou d'une `Map` Java), `set` (non ordonné, valeurs uniques).

```python
fruits = ["pomme", "banane", "kiwi"]   # list
point = (3, 4)                          # tuple, immuable
person = {"name": "Max", "age": 30}       # dict
unique_tags = {"php", "python", "php"}      # set → {"php", "python"}

fruits.append("mangue")
person["email"] = "max@example.com"
```

**Bonne pratique** : préférer un `tuple` à une `list` pour une donnée dont la structure ne doit pas changer (coordonnées, paire clé-valeur retournée) — l'immutabilité communique l'intention.

### 07 - Chaînes de caractères et f-strings

**Explication** — `str` est immuable (comme en PHP/Java). Les **f-strings** (Python 3.6+) sont la façon idiomatique moderne d'interpoler des variables — préférées à `.format()` ou à la concaténation `+`.

```python
name = "Max"
age = 30

greeting = f"Bonjour {name}, tu as {age} ans."
greeting2 = f"Dans 5 ans : {age + 5} ans."   # expressions arbitraires autorisées dans {}

multiline = """
Texte
sur plusieurs lignes
"""
```

**Erreur fréquente** : utiliser `%`-formatting ou `.format()` par habitude d'un vieux tutoriel — les f-strings sont plus lisibles et légèrement plus rapides, à privilégier systématiquement en code moderne.

### 08 - Fonctions, arguments par défaut et *args/**kwargs

**Explication** — `def` déclare une fonction. Les paramètres peuvent avoir une valeur par défaut, être nommés à l'appel, ou capturer un nombre variable d'arguments positionnels (`*args`) ou nommés (`**kwargs`).

```python
def greet(name, greeting="Bonjour"):
    return f"{greeting}, {name} !"

greet("Max")                       # "Bonjour, Max !"
greet("Max", greeting="Salut")       # argument nommé, "Salut, Max !"

def summarize(*args, **kwargs):
    print(args)     # tuple des positionnels
    print(kwargs)      # dict des nommés

summarize(1, 2, 3, name="Max")  # (1, 2, 3)  {'name': 'Max'}
```

**Erreur fréquente** : utiliser un objet mutable (`list`, `dict`) comme valeur par défaut d'un paramètre — cet objet est créé **une seule fois** au chargement de la fonction et partagé entre tous les appels, un piège classique.

```python
def append_item(item, items=[]):  # PIÈGE : la même liste persiste entre les appels
    items.append(item)
    return items

def append_item(item, items=None):  # correct
    if items is None:
        items = []
    items.append(item)
    return items
```

### 09 - Modules et packages

**Explication** — Un fichier `.py` est un **module**. Un dossier contenant un `__init__.py` (optionnel depuis Python 3.3 pour les *namespace packages*, mais conseillé) est un **package** — équivalent conceptuel des namespaces PSR-4 en PHP.

```python
# fichier math_utils.py
def square(x):
    return x * x

# fichier main.py
from math_utils import square
import math_utils as mu

print(square(4))       # 16
print(mu.square(4))    # 16
```

**Bonne pratique** : éviter `from module import *` (importe tout sans préciser les noms) — pollue l'espace de noms et rend l'origine d'une fonction difficile à tracer, à l'inverse d'un import explicite.

## 4. Concepts intermédiaires 🟡

- **POO** : `class`, `self` (équivalent explicite de `$this`, mais passé en premier paramètre de chaque méthode plutôt qu'implicite), `__init__` (constructeur), héritage simple et multiple, méthodes spéciales *dunder* (`__str__`, `__eq__`, `__len__`...) qui permettent à un objet de s'intégrer aux opérateurs natifs du langage.

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f"User({self.name}, {self.age} ans)"

    def __eq__(self, other):
        return self.name == other.name and self.age == other.age

u = User("Max", 30)
print(u)  # utilise __str__ automatiquement : "User(Max, 30 ans)"
```

- **Compréhensions** : syntaxe concise pour construire `list`/`dict`/`set` à partir d'un itérable — équivalent fonctionnel d'un `array_map`/`array_filter` PHP condensé en une expression.

```python
squares = [x**2 for x in range(10)]                  # list comprehension
evens_only = [x for x in range(10) if x % 2 == 0]      # avec filtre
word_lengths = {w: len(w) for w in ["php", "python"]}    # dict comprehension
```

- **Décorateurs** : fonctions qui enveloppent une autre fonction pour en modifier le comportement sans toucher son code — proche des attributs PHP 8 (`#[Route]`) ou des annotations Java (`@Override`) dans l'intention, mais implémentés en pur Python (ce sont des fonctions ordinaires).

```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"Appel de {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_call
def add(a, b):
    return a + b

add(2, 3)  # affiche "Appel de add" puis retourne 5
```

- **Gestion des exceptions** : `try/except/else/finally`, hiérarchie d'exceptions natives (`ValueError`, `TypeError`, `KeyError`...), exceptions custom héritant d'`Exception`. Contrairement à Java, il n'existe pas de distinction *checked/unchecked* — toute exception peut se propager librement sans déclaration obligatoire.

```python
class InsufficientFundsError(Exception):
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError("Solde insuffisant")
    return balance - amount

try:
    withdraw(100, 150)
except InsufficientFundsError as e:
    print(f"Erreur : {e}")
finally:
    print("Opération terminée")
```

- **Context managers (`with`)** : garantissent qu'une ressource (fichier, connexion) est proprement libérée même en cas d'exception — équivalent du `try-with-resources` Java ou d'un `finally` implicite.

```python
with open("data.txt") as f:
    content = f.read()
# le fichier est automatiquement fermé ici, même si une exception survient dans le bloc
```

- **Itérateurs et générateurs** : `yield` transforme une fonction en générateur, qui produit ses valeurs paresseusement (une à la fois, à la demande) plutôt que de construire une collection entière en mémoire.

```python
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

for num in fibonacci(10):  # ne calcule que ce qui est consommé
    print(num)
```

- **Environnements virtuels et gestion de dépendances** : `venv` isole les dépendances d'un projet (équivalent du principe de `vendor/` local en PHP, mais au niveau de l'interpréteur entier). `pip` installe depuis `requirements.txt` (approche historique) ou `pyproject.toml` (standard moderne, équivalent de `composer.json`).

```bash
python3 -m venv .venv       # crée l'environnement virtuel
source .venv/bin/activate     # active (Linux/macOS)
pip install requests            # installe une dépendance
pip freeze > requirements.txt     # fige les versions installées
```

## 5. Concepts avancés 🟠🔴

- **Le GIL (Global Interpreter Lock)** : en CPython, un verrou global garantit qu'**un seul thread** exécute du bytecode Python à la fois, même sur une machine multi-cœurs. Conséquence : le multithreading classique (`threading`) n'accélère pas du code CPU-bound (calcul pur), mais reste utile pour de l'I/O-bound (attente réseau/disque, le GIL est relâché pendant l'attente).

```python
import threading

def cpu_bound_task():
    total = sum(i * i for i in range(10**7))

threads = [threading.Thread(target=cpu_bound_task) for _ in range(4)]
# 4 threads ici ne vont PAS diviser le temps par 4 : le GIL sérialise l'exécution
```

- **Multiprocessing** : pour du parallélisme CPU réel, `multiprocessing` lance des **processus** séparés (chacun avec son propre interpréteur et son propre GIL) plutôt que des threads — coût de communication inter-processus plus élevé, mais parallélisme effectif.

```python
from multiprocessing import Pool

def square(x):
    return x * x

with Pool(4) as pool:
    results = pool.map(square, range(10))  # réparti sur 4 processus réels
```

- **Asyncio (async/await)** : modèle de concurrence coopérative à event loop unique, pour du code I/O-bound à haute concurrence (nombreuses requêtes réseau simultanées) — conceptuellement très proche de l'event loop de Node.js ([`../nodejs/`](../nodejs/)), mais explicite (`async def`/`await` obligatoires à chaque point de suspension, alors que Node.js s'appuie sur des callbacks/Promises implicites dans tout le runtime).

```python
import asyncio

async def fetch_data(delay):
    await asyncio.sleep(delay)  # suspend cette coroutine, l'event loop traite autre chose entre-temps
    return "data"

async def main():
    results = await asyncio.gather(fetch_data(1), fetch_data(2))
    print(results)

asyncio.run(main())
```

- **Typage statique optionnel (`mypy`)** : les type hints (section 04) ne sont vérifiés par rien nativement — l'outil `mypy` analyse le code statiquement et signale les incohérences de type avant l'exécution, apportant une garantie proche (mais opt-in et externe) du typage statique natif de Java.
- **Performance et alternatives à CPython** : PyPy (implémentation avec JIT, souvent 4-10x plus rapide sur du code pur Python), Cython (compile du Python annoté en C), extensions C natives (NumPy délègue l'essentiel du calcul lourd à du code C compilé) — la stratégie courante en Python n'est pas "réécrire le langage" mais "déléguer les parties critiques à du code natif".
- **Métaclasses et descripteurs** : mécanismes de très bas niveau permettant de personnaliser la création de classes elles-mêmes (`type` est la métaclasse par défaut) ou le comportement d'accès aux attributs (`__get__`/`__set__`) — utilisés par des frameworks (Django ORM s'en sert pour ses champs de modèle) mais rarement nécessaires en code applicatif courant.

## 6. Commandes / syntaxe à connaître

```bash
python3 --version                 # version de l'interpréteur
python3 script.py                  # exécuter un script
python3 -m venv .venv                # créer un environnement virtuel
source .venv/bin/activate              # activer l'environnement (Linux/macOS)
pip install <package>                    # installer une dépendance
pip freeze > requirements.txt              # figer les dépendances installées
python3 -m pytest                            # lancer les tests (pytest, standard de facto)
python3 -m mypy script.py                      # vérification de types statique
```

```python
# Syntaxe essentielle à avoir sous les doigts
[x for x in items if condition]        # list comprehension
{k: v for k, v in items}                 # dict comprehension
with open("f.txt") as f: ...               # context manager
try / except Exception as e: ...             # gestion d'exception
async def f(): await other()                   # coroutine
def f(*args, **kwargs): ...                      # arguments variadiques
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Gestionnaire de tâches en CLI, orienté objet**

Construire une petite application Python en ligne de commande (sans framework) qui doit :
- Modéliser une tâche avec une `class Task` (`title`, `done`, méthode `__str__` pour un affichage lisible).
- Stocker les tâches dans une `list[Task]` en mémoire, avec des opérations via des compréhensions (filtrer les tâches en cours, terminées).
- Utiliser un générateur (`yield`) pour parcourir les tâches en retard sans construire de liste intermédiaire.
- Gérer une exception métier custom `TaskNotFoundError`, levée et catchée proprement.
- Persister les tâches en JSON entre deux exécutions (`json.dump`/`json.load`), avec un `with open(...)` pour la lecture/écriture de fichier.

Objectif : mobiliser POO, compréhensions, générateurs, gestion d'exceptions et manipulation de fichiers dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux (types dynamiques, indentation, structures de contrôle)
- [ ] Savoir créer et activer un environnement virtuel (`venv`)
- [ ] Maîtriser la syntaxe principale (POO, compréhensions, fonctions)
- [ ] Comprendre les concepts importants (décorateurs, générateurs, context managers)
- [ ] Savoir debugger (`pdb`, breakpoints IDE, stack traces)
- [ ] Connaître les bonnes pratiques (f-strings, éviter les mutables par défaut, PEP 8)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (GIL, asyncio, multiprocessing, typage statique optionnel)

## 10. Ressources

- [Documentation officielle Python](https://docs.python.org/3/) — référence complète du langage et de la bibliothèque standard.
- [PEP 8 — Style Guide for Python Code](https://peps.python.org/pep-0008/) — conventions de style officielles.
- [Real Python](https://realpython.com/) — tutoriels et explications de qualité sur l'écosystème Python.
- [mypy — documentation](https://mypy.readthedocs.io/) pour le typage statique optionnel.
- [roadmap.sh — Python](https://roadmap.sh/python) — vue d'ensemble du parcours d'apprentissage.
