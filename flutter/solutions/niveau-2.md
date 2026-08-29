# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```dart
class FruitList extends StatelessWidget {
  final List<String> fruits = ['Pomme', 'Banane', 'Cerise'];

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: fruits.length,
      itemBuilder: (context, index) => ListTile(title: Text(fruits[index])),
    );
  }
}
```

## Exercice 2

```dart
class UserListScreen extends StatelessWidget {
  const UserListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text('Utilisateur 42'),
      onTap: () => Navigator.push(
        context,
        MaterialPageRoute(builder: (context) => UserDetailScreen(userId: '42')),
      ),
    );
  }
}

class UserDetailScreen extends StatelessWidget {
  final String userId;
  const UserDetailScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context) => Scaffold(body: Text('Détail utilisateur $userId'));
}
```

## Exercice 3

```dart
Future<String> fetchGreeting() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Bonjour depuis le serveur';
}

class GreetingWidget extends StatelessWidget {
  const GreetingWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: fetchGreeting(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return CircularProgressIndicator();
        }
        return Text(snapshot.data ?? 'Erreur');
      },
    );
  }
}
```

## Exercice 4

```dart
class EmailForm extends StatelessWidget {
  EmailForm({super.key});
  final formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: formKey,
      child: Column(
        children: [
          TextFormField(
            validator: (value) {
              if (value == null || value.isEmpty) return 'Champ requis';
              if (!value.contains('@')) return 'Email invalide';
              return null;
            },
          ),
          ElevatedButton(
            onPressed: () {
              if (formKey.currentState!.validate()) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Formulaire valide')),
                );
              }
            },
            child: Text('Valider'),
          ),
        ],
      ),
    );
  }
}
```

## Exercice 5

```dart
class CounterModel extends ChangeNotifier {
  int count = 0;
  void increment() {
    count++;
    notifyListeners();
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: MyApp(),
    ),
  );
}

// Widget d'affichage, consomme sans recevoir l'état en paramètre :
class CounterDisplay extends StatelessWidget {
  const CounterDisplay({super.key});
  @override
  Widget build(BuildContext context) {
    final count = context.watch<CounterModel>().count;
    return Text('Compteur : $count');
  }
}

// Widget d'incrémentation, séparé, sans lien direct avec CounterDisplay :
class IncrementButton extends StatelessWidget {
  const IncrementButton({super.key});
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.read<CounterModel>().increment(),
      child: Text('+1'),
    );
  }
}
```
