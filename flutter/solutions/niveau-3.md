# Solutions — Niveau 3 (Avancé)

## Exercice 1

```dart
class ParentWithTimer extends StatefulWidget {
  const ParentWithTimer({super.key});
  @override
  State<ParentWithTimer> createState() => _ParentWithTimerState();
}

class _ParentWithTimerState extends State<ParentWithTimer> {
  int seconds = 0;

  @override
  void initState() {
    super.initState();
    Timer.periodic(Duration(seconds: 1), (_) => setState(() => seconds++));
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Secondes : $seconds'),
        const Text('Titre fixe'),  // const : ne sera jamais reconstruit
      ],
    );
  }
}
```

Quand `build()` est ré-exécuté suite à `setState`, Flutter compare chaque widget enfant à sa version précédente. Un widget `const` produit une **instance identique** (même référence en mémoire) à chaque appel de `build()`, car ses paramètres sont fixés à la compilation — Flutter détecte cette identité et saute entièrement le travail de reconstruction/re-render de ce sous-arbre, même si le parent se redessine à chaque tick du timer.

## Exercice 2

En mode debug (JIT), le code Dart est compilé "à la volée" pendant que l'app tourne, ce qui permet d'injecter du code modifié dans la VM Dart en cours d'exécution sans redémarrer l'app entière — c'est cette VM Dart persistante et le compilateur JIT qui rendent le hot reload possible. En mode release (AOT), tout le code Dart est compilé une fois pour toutes en code machine natif avant l'exécution : il n'y a plus de VM Dart interprétant du bytecode ni de compilateur JIT actif au runtime, donc pas de mécanisme pour injecter du code à chaud — mais en contrepartie, l'app démarre plus vite (aucune compilation à faire au lancement) et s'exécute de façon plus prévisible en performance.

## Exercice 3

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  void increment() => state = state + 1;
}

final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) => CounterNotifier());

// Consommation dans un ConsumerWidget, sans BuildContext.watch :
class CounterDisplay extends ConsumerWidget {
  const CounterDisplay({super.key});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('Compteur : $count');
  }
}
```

Le principal avantage pratique de Riverpod est qu'il ne dépend pas du `BuildContext` pour accéder à l'état (contrairement à `Provider.of(context)`/`context.watch`) — les providers Riverpod sont des objets globaux indépendants de l'arbre de widgets, ce qui les rend testables unitairement sans monter de widget, et évite des erreurs classiques de Provider liées à un `BuildContext` invalide (utilisé hors de l'arbre, ou avant le premier `build()`).

## Exercice 4

Côté Dart, on crée un `MethodChannel` avec un nom unique (ex. `'com.example.app/battery'`) et on appelle `channel.invokeMethod('getBatteryLevel')`, qui retourne un `Future` en attente de réponse native. Côté natif (Swift pour iOS, Kotlin pour Android), on enregistre un `MethodChannel` du même nom dans le code de démarrage de l'app (`AppDelegate`/`MainActivity`), et on implémente un handler qui reçoit l'appel `'getBatteryLevel'`, lit la vraie valeur via l'API système native (`UIDevice.current.batteryLevel` sur iOS), puis renvoie le résultat au Dart via `result(batteryLevel)`. La communication passe par un pont de sérialisation standard (proche du bridge historique React Native) : c'est un mécanisme ponctuel pour des appels occasionnels, pas destiné à un échange à haute fréquence comme le rendu par frame.
