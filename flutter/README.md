# Flutter

## 1. Introduction

Flutter est un framework mobile (et multi-plateforme : web, desktop) créé par Google, basé sur le langage **Dart**. Contrairement à React Native ([`../react-native/`](../react-native/)) qui pilote de vrais composants natifs via un pont/JSI, Flutter **dessine lui-même chaque pixel** avec son propre moteur de rendu (Skia/Impeller) — l'app ne dépend quasiment pas des composants UI natifs de la plateforme. Ce dossier couvre Dart et Flutter ensemble, Dart n'ayant pas de dossier dédié dans cette roadmap.

**À quoi sert-il ?**
- Construire une application mobile pour iOS et Android (et web/desktop) à partir d'une seule base de code, avec un rendu visuellement identique sur toutes les plateformes.
- Créer des UI riches et hautement personnalisées où le contrôle pixel-perfect du rendu prime.

**Où se situe-t-il dans une architecture mobile ?** Dart est compilé en code natif ARM/x64 (AOT — Ahead-Of-Time — en production), et Flutter dessine l'intégralité de l'interface sur un canvas géré par son propre moteur de rendu, plutôt que d'assembler des composants UI natifs comme React Native. Il n'y a donc pas de "pont" JS/natif à traverser à chaque frame — un facteur clé de sa fluidité.

**Avantages**
- Rendu identique pixel pour pixel sur iOS/Android (et au-delà) : pas de divergence visuelle liée aux composants natifs de chaque plateforme, contrairement à React Native.
- Performance proche du natif : Dart compilé en AOT, pas de pont à traverser pour le rendu (contrairement à l'ancien pont React Native).
- Hot reload très rapide, riche bibliothèque de widgets Material Design et Cupertino (style iOS) prêts à l'emploi.

**Limites**
- Dart est un langage à apprendre en plus (contrairement à React Native qui réutilise du JS/TS déjà connu) — coût d'entrée plus élevé si l'équipe vient du web.
- Parce que Flutter ne s'appuie pas sur les composants natifs, une évolution de design system natif (iOS/Android) n'est pas automatiquement répercutée — Flutter doit la reproduire lui-même.
- Taille d'app généralement plus importante (le moteur de rendu et le framework sont embarqués) comparé à une app native pure.

## 2. Prérequis

- POO déjà pratiquée dans un autre langage (PHP, Java, Python) — les concepts de classes, héritage, interfaces sont directement transférables à Dart.
- SDK Flutter installé (`flutter doctor` pour vérifier la configuration), Android Studio ou Xcode pour les émulateurs/simulateurs.
- Aucun prérequis JavaScript nécessaire (contrairement à React Native) — Dart est un langage à part entière, couvert dans la section 3 ci-dessous.

## 3. Rappel des bases 🟢

### 01 - Dart : syntaxe et typage

**Explication** — Dart est un langage compilé, orienté objet, à **typage statique fort** avec inférence (`var` infère le type à la déclaration, comme `var` en Java) — plus proche de Java/TypeScript que du JavaScript/Python dynamique.

```dart
void main() {
  String name = 'Max';
  var age = 30;              // inféré en int, reste int pour toujours
  final city = 'Paris';       // assignable une seule fois (comme const en JS)
  const pi = 3.14159;           // constante à la compilation

  print('Bonjour $name, tu as $age ans.');  // interpolation de chaîne avec $
}
```

**Erreur fréquente** : confondre `final` et `const` — `final` est assignable une seule fois mais peut dépendre d'une valeur connue seulement à l'exécution ; `const` doit être connue à la **compilation** (une constante littérale ou dérivée d'autres constantes).

### 02 - Null safety

**Explication** — Depuis Dart 2.12, le typage distingue explicitement les types nullable (`String?`) des non-nullable (`String`) — le compilateur **interdit** d'assigner `null` à un type non-nullable, éliminant à la compilation toute une classe d'erreurs `NullPointerException` runtime.

```dart
String name = 'Max';       // ne peut jamais être null
String? nickname;           // peut être null, doit être vérifié avant usage

// print(nickname.length);  // erreur de compilation : nickname pourrait être null
print(nickname?.length);     // opérateur ?. : null-safe, retourne null si nickname est null
print(nickname ?? 'Anonyme');  // opérateur ?? : valeur par défaut si null
```

**Bonne pratique** : éviter l'opérateur `!` (force un type nullable à être traité comme non-null) sauf certitude absolue — il réintroduit le risque d'exception que la null safety cherche justement à éliminer.

### 03 - Structures de contrôle et collections

**Explication** — `if/else`, `for`, `while` très proches de Java/PHP. `List`, `Map`, `Set` sont les collections natives, avec un support natif de la syntaxe *collection for* et *collection if* pour les construire de façon concise.

```dart
List<int> scores = [12, 15, 8, 18, 10];
Map<String, int> ages = {'Max': 30, 'Alice': 25};

List<int> evens = [for (var s in scores) if (s.isEven) s];  // collection for + if
```

**Cas d'usage** : la *collection for* est très utilisée directement dans la construction de listes de widgets (voir section 04), pour générer dynamiquement une liste d'éléments UI sans `.map().toList()` explicite.

### 04 - Tout est un widget

**Explication** — Le concept central de Flutter : **toute l'UI est un arbre de widgets** — texte, marge, couleur, layout sont chacun un widget, composés les uns dans les autres plutôt qu'exprimés via des propriétés CSS séparées.

```dart
Widget build(BuildContext context) {
  return Container(
    padding: EdgeInsets.all(16),
    color: Colors.white,
    child: Text('Bonjour, Max !', style: TextStyle(fontSize: 18)),
  );
}
```

**Différence clé avec React/Vue** : il n'y a pas de langage de template séparé (JSX, templates HTML) — l'arbre de widgets est construit avec du Dart pur, imbriqué directement dans le code.

### 05 - StatelessWidget vs StatefulWidget

**Explication** — Un `StatelessWidget` n'a pas d'état interne mutable, il se redessine uniquement si ses paramètres changent (comme un composant fonctionnel React sans `useState`). Un `StatefulWidget` sépare la définition du widget (immuable) de son `State` associé (mutable), qui peut appeler `setState()` pour déclencher un redessin.

```dart
class Greeting extends StatelessWidget {
  final String name;
  const Greeting({super.key, required this.name});

  @override
  Widget build(BuildContext context) => Text('Bonjour, $name !');
}

class Counter extends StatefulWidget {
  const Counter({super.key});
  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int count = 0;

  void _increment() => setState(() => count++);  // déclenche un redessin

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(onPressed: _increment, child: Text('Compteur : $count'));
  }
}
```

**Erreur fréquente** : modifier une variable d'état sans passer par `setState()` — la valeur change bien en mémoire, mais Flutter ne sait pas qu'il doit redessiner le widget, l'UI reste figée à l'ancienne valeur affichée.

### 06 - Widgets de layout

**Explication** — `Row` (horizontal) et `Column` (vertical) sont les équivalents Flexbox de Flutter, `Container` combine padding/marge/couleur/taille (proche d'une `div` stylée), `Expanded`/`Flexible` distribuent l'espace disponible entre enfants d'un `Row`/`Column`.

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [
    Icon(Icons.star),
    Expanded(child: Text('Titre qui prend l\'espace restant')),
    Icon(Icons.arrow_forward),
  ],
)
```

**Bonne pratique** : utiliser `Expanded` pour éviter l'erreur classique "RenderFlex overflowed" quand un enfant (souvent un `Text` long) dépasse l'espace disponible d'un `Row`.

### 07 - Hot reload

**Explication** — Flutter injecte le code modifié dans la VM Dart en cours d'exécution sans perdre l'état de l'application (contrairement à un rechargement complet) — itération quasi instantanée, un atout de développement majeur.

```bash
flutter run    # lance l'app, hot reload actif automatiquement (touche 'r' pour forcer)
```

**Cas d'usage** : ajuster un padding, une couleur, ou le texte d'un widget se répercute en moins d'une seconde sans perdre la position de navigation actuelle dans l'app — précieux pour l'ajustement fin de l'UI.

## 4. Concepts intermédiaires 🟡

- **Gestion d'état avec setState à l'échelle** : `setState` suffit pour un état local simple (section 05), mais devient rapidement limitant dès que plusieurs widgets éloignés dans l'arbre doivent partager un état — d'où le recours à des solutions dédiées (`Provider`, `Riverpod`) plutôt que de faire remonter l'état manuellement à travers de nombreux niveaux de widgets.

```dart
// Avec Provider : exposer un état partagé sans le faire descendre manuellement à chaque niveau
class CartModel extends ChangeNotifier {
  final List<Item> _items = [];
  List<Item> get items => _items;
  void add(Item item) { _items.add(item); notifyListeners(); }
}

// Consommation dans un widget descendant, sans prop drilling :
Consumer<CartModel>(builder: (context, cart, child) => Text('${cart.items.length} articles'));
```

- **Navigation (Navigator)** : Flutter gère nativement une pile de routes (`Navigator.push`/`pop`), comparable conceptuellement à un Stack Navigator de React Navigation, mais intégré au framework plutôt qu'une librairie tierce.

```dart
Navigator.push(context, MaterialPageRoute(builder: (context) => DetailScreen(id: item.id)));
Navigator.pop(context);  // retour à l'écran précédent
```

- **Programmation asynchrone (Future, async/await)** : syntaxe très proche de JavaScript/Python — `Future<T>` représente une valeur disponible plus tard, `async`/`await` pour l'attendre de façon lisible.

```dart
Future<String> fetchUserName() async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return jsonDecode(response.body)['name'];
}
```

- **FutureBuilder** : widget qui reconstruit son contenu selon l'état d'un `Future` (en attente, résolu, en erreur) — pattern déclaratif pour afficher un état de chargement sans gérer manuellement un booléen `isLoading`.

```dart
FutureBuilder<String>(
  future: fetchUserName(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) return CircularProgressIndicator();
    if (snapshot.hasError) return Text('Erreur');
    return Text(snapshot.data!);
  },
)
```

- **Formulaires** : `Form` + `TextFormField` avec des `validator` déclaratifs, `GlobalKey<FormState>` pour déclencher la validation globale — équivalent conceptuel des Reactive Forms Angular ou des libs de formulaires React.

```dart
final formKey = GlobalKey<FormState>();

Form(
  key: formKey,
  child: TextFormField(
    validator: (value) => (value == null || value.isEmpty) ? 'Champ requis' : null,
  ),
);

// Déclenchement : formKey.currentState!.validate();
```

## 5. Concepts avancés 🟠🔴

- **Compilation AOT vs JIT** : en développement, Dart utilise la compilation JIT (Just-In-Time, permet le hot reload) ; en production (`flutter build`), le code est compilé en AOT (Ahead-Of-Time) vers du code machine natif — démarrage plus rapide et performance prévisible, sans interprétation ni compilation à la volée au runtime.
- **Widgets const et performance de reconstruction** : marquer un widget `const` quand ses paramètres sont connus à la compilation permet à Flutter de **réutiliser l'instance** plutôt que d'en recréer une identique à chaque `build()` — optimisation directe, à appliquer systématiquement dès que possible.

```dart
// Sans const : une nouvelle instance de Text est recréée à chaque reconstruction du parent
Text('Titre fixe')

// Avec const : Flutter réutilise l'instance existante, évite un travail de diff inutile
const Text('Titre fixe')
```

- **Gestion d'état à l'échelle (Riverpod/Bloc)** : au-delà de `Provider` (section intermédiaire), des architectures plus structurées existent — Riverpod (successeur de Provider, sans dépendance au `BuildContext`, testable plus facilement) ou Bloc (séparation stricte events → state, discipline proche de Redux) — le choix dépend de la taille et de la rigueur architecturale souhaitée pour le projet.
- **Platform channels** : mécanisme permettant à Dart d'appeler du code natif (Swift/Kotlin) pour accéder à une API non couverte par un package Flutter existant — équivalent conceptuel des modules natifs custom de React Native.
- **Performance et profiling** : Flutter DevTools permet de visualiser l'arbre de widgets, détecter les reconstructions inutiles (`build()` appelé trop souvent), et profiler les frames qui dépassent le budget de 16ms (60fps) — discipline similaire au profiling React (React DevTools Profiler), avec un outillage dédié au rendu propre à Flutter.

## 6. Commandes / syntaxe à connaître

```bash
flutter doctor                 # vérifie la configuration de l'environnement
flutter create mon_app           # créer un nouveau projet
flutter run                        # lancer l'app (hot reload actif)
flutter build apk                    # build Android de production
flutter build ios                      # build iOS de production
flutter test                             # lancer les tests
```

```dart
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});
  @override
  Widget build(BuildContext context) => Container(child: Text('...'));
}

setState(() { /* mutation d'état */ });
Navigator.push(context, MaterialPageRoute(builder: (context) => NextScreen()));
FutureBuilder(future: ..., builder: (context, snapshot) => ...);
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Application de notes avec navigation et état partagé**

Construire une petite application Flutter qui doit :
- Un écran `NoteListScreen` (`StatelessWidget`) affichant une liste de notes via `ListView.builder` (équivalent de `FlatList`).
- Un modèle d'état `NotesModel extends ChangeNotifier` exposé via `Provider`, avec les méthodes `addNote`/`removeNote`.
- Un écran `NoteFormScreen` avec un `Form` validé (`TextFormField` + `validator`) pour ajouter une note, accessible via `Navigator.push`.
- Chaque note de la liste navigue vers un écran de détail avec `Navigator.push` et l'id de la note en paramètre.
- Bonus : charger une liste d'utilisateurs depuis une API publique avec `http` + `FutureBuilder`, en affichant un `CircularProgressIndicator` pendant le chargement.

Objectif : mobiliser widgets stateless/stateful, layout, navigation, gestion d'état partagé (Provider) et formulaires dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux de Dart (typage, null safety, collections)
- [ ] Comprendre le concept de widget et l'arbre de widgets
- [ ] Savoir créer un projet Flutter et utiliser le hot reload
- [ ] Maîtriser la syntaxe principale (StatelessWidget/StatefulWidget, layout Row/Column)
- [ ] Comprendre les concepts importants (Navigator, Future/async, FutureBuilder, formulaires)
- [ ] Savoir debugger avec Flutter DevTools
- [ ] Connaître les bonnes pratiques (widgets const, setState localisé, Expanded pour éviter les overflows)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (AOT vs JIT, Riverpod/Bloc, platform channels)

## 10. Ressources

- [Documentation officielle Flutter](https://docs.flutter.dev) — référence complète des widgets et du framework.
- [Documentation officielle Dart](https://dart.dev/guides) — référence du langage.
- [Widget catalog Flutter](https://docs.flutter.dev/ui/widgets) — catalogue de tous les widgets disponibles.
- [Provider — package pub.dev](https://pub.dev/packages/provider) et [Riverpod](https://riverpod.dev/) pour la gestion d'état.
- [roadmap.sh — Flutter](https://roadmap.sh/flutter) — vue d'ensemble du parcours d'apprentissage.
