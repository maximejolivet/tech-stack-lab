# Exercices Flutter — Niveau 1 (Bases)

## Exercice 1 — Hello, Dart

Écris une fonction `main()` qui déclare une variable `String name`, une variable `var age` (inférée en `int`), et affiche `"Bonjour $name, tu as $age ans."` avec l'interpolation de chaîne.

## Exercice 2 — Null safety

Sans exécuter le code, prédis ce qui se passe, puis corrige-le pour qu'il compile :

```dart
void main() {
  String? nickname;
  print(nickname.length);
}
```

## Exercice 3 — StatelessWidget

Crée un widget `Greeting extends StatelessWidget` qui reçoit un paramètre `name` (requis) et affiche `Text('Bonjour, $name !')`.

## Exercice 4 — Compteur (StatefulWidget)

Crée un widget `Counter extends StatefulWidget` avec un `State` qui maintient `int count = 0`, un `ElevatedButton` qui incrémente via `setState`, et affiche la valeur courante.

## Exercice 5 — Layout Row/Column

Crée un widget qui affiche trois `Icon` dans un `Row` avec `mainAxisAlignment: MainAxisAlignment.spaceAround`, puis enveloppe-le dans une `Column` avec un `Text` de titre au-dessus.
