# Solutions — Niveau 3

## 3.1 — Asyncio

```python
import asyncio
import time

async def fetch(delay, label):
    await asyncio.sleep(delay)
    return f"{label} terminé après {delay}s"

async def main():
    start = time.perf_counter()
    results = await asyncio.gather(
        fetch(1, "requete-1"),
        fetch(2, "requete-2"),
        fetch(3, "requete-3"),
    )
    elapsed = time.perf_counter() - start
    print(results)
    print(f"Temps total : {elapsed:.2f}s")  # proche de 3s, pas de 6s (1+2+3)

asyncio.run(main())
```

## 3.2 — GIL et multiprocessing

```python
import time
from multiprocessing import Pool

def cpu_bound(n):
    return sum(i * i for i in range(n))

N = 10**7

# Séquentiel
start = time.perf_counter()
results = [cpu_bound(N) for _ in range(4)]
print(f"Séquentiel : {time.perf_counter() - start:.2f}s")

# Multiprocessing
start = time.perf_counter()
with Pool(4) as pool:
    results = pool.map(cpu_bound, [N] * 4)
print(f"Multiprocessing : {time.perf_counter() - start:.2f}s")
```

Avec `threading.Thread`, les 4 threads se disputeraient le même GIL : un seul exécute du bytecode Python à un instant donné, donc un calcul CPU-bound pur ne serait pas accéléré (le temps total resterait proche du séquentiel, avec en plus l'overhead de changement de contexte entre threads). `multiprocessing` contourne le problème en lançant 4 **processus** séparés, chacun avec son propre interpréteur et son propre GIL, permettant un vrai parallélisme sur 4 cœurs.

## 3.3 — Context manager custom

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(label):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label} : {elapsed:.4f}s")

with timer("bloc critique"):
    total = sum(i for i in range(10**6))

# Le `finally` garantit l'affichage même si une exception est levée dans le bloc :
try:
    with timer("bloc avec erreur"):
        raise ValueError("boom")
except ValueError:
    print("Exception catchée après le timer")
```

## 3.4 — Typage statique avec mypy

```python
from typing import Callable

def process(items: list[int], transform: Callable[[int], int]) -> list[int]:
    return [transform(item) for item in items]

result = process([1, 2, 3], lambda x: x * 2)  # OK pour mypy

# Incohérence volontaire :
result_bad = process([1, 2, 3], lambda x: str(x))  
# mypy signale : Argument "transform" has incompatible type
# "Callable[[int], str]"; expected "Callable[[int], int]"
```
