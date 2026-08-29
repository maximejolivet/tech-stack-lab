# Solutions — Niveau 1 (Bases)

## Exercice 1

```dart
void main() {
  String name = 'Max';
  var age = 30;
  print('Bonjour $name, tu as $age ans.');
}
```

## Exercice 2

```dart
// ❌ Erreur de compilation : nickname est String?, .length ne peut pas être
// appelé directement sans vérifier ou gérer le cas null.
void main() {
  String? nickname;
  print(nickname.length);
}

// ✅ Corrigé, avec l'opérateur null-safe ?.
void main() {
  String? nickname;
  print(nickname?.length);      // affiche "null" si nickname est null
  print(nickname?.length ?? 0);   // affiche 0 par défaut si nickname est null
}
```

## Exercice 3

```dart
class Greeting extends StatelessWidget {
  final String name;
  const Greeting({super.key, required this.name});

  @override
  Widget build(BuildContext context) => Text('Bonjour, $name !');
}
```

## Exercice 4

```dart
class Counter extends StatefulWidget {
  const Counter({super.key});
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int count = 0;

  void _increment() => setState(() => count++);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Compteur : $count'),
        ElevatedButton(onPressed: _increment, child: Text('+1')),
      ],
    );
  }
}
```

## Exercice 5

```dart
class IconRow extends StatelessWidget {
  const IconRow({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Mes icônes'),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: [
            Icon(Icons.star),
            Icon(Icons.favorite),
            Icon(Icons.home),
          ],
        ),
      ],
    );
  }
}
```
