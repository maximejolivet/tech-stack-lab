# Solutions — Niveau 2

## 2.1 — POO et dunder methods

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def __str__(self):
        return f"Rectangle({self.width}x{self.height})"

    def __eq__(self, other):
        return self.width == other.width and self.height == other.height

r1 = Rectangle(3, 4)
r2 = Rectangle(3, 4)
print(r1)          # Rectangle(3x4)
print(r1 == r2)     # True
```

## 2.2 — Compréhensions

```python
names = ["Max", "Alice", "Bob", "Charlotte"]

uppercased = [n.upper() for n in names]
long_names = [n for n in names if len(n) > 4]
lengths = {n: len(n) for n in names}
```

## 2.3 — Décorateur de chronométrage

```python
import time
from functools import wraps

def timed(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} exécuté en {elapsed:.4f}s")
        return result
    return wrapper

@timed
def slow_loop():
    total = sum(i for i in range(10**7))
    return total

slow_loop()
```

## 2.4 — Générateur

```python
def even_numbers(limit):
    for n in range(limit + 1):
        if n % 2 == 0:
            yield n

for n in even_numbers(10):
    print(n)
```

## 2.5 — Exception custom

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
```
