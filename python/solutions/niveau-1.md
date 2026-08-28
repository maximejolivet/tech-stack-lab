# Solutions — Niveau 1

## 1.1 — Hello, dynamique

```python
name = "Max"
age = 30
print(f"Bonjour {name}, tu as {age} ans.")
```

## 1.2 — Coercition piège

```python
print("5" + "3")     # "53" (concaténation de strings)
print(int("5") + 3)  # 8 (conversion explicite)
print("5" + 3)        # TypeError: can only concatenate str (not "int") to str
```

Contrairement à PHP/JS qui coercent silencieusement les types dans un `+` mixte, Python est **typé fort** : additionner une `str` et un `int` sans conversion explicite lève une `TypeError` plutôt que de deviner une conversion.

## 1.3 — Liste et boucle

```python
scores = [12, 15, 8, 18, 10]
total = 0
for score in scores:
    total += score
average = total / len(scores)
print(f"Moyenne : {average}")
```

## 1.4 — Dict et f-string

```python
person = {"name": "Max", "age": 30}
print(f"{person['name']} a {person['age']} ans")
```

## 1.5 — Match statement

```python
def day_type(day):
    match day:
        case 6 | 7:
            return "Week-end"
        case _:
            return "Semaine"
```
