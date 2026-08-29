# Exercices Flutter — Niveau 2 (Intermédiaire)

## Exercice 1 — ListView.builder

Étant donné une `List<String> fruits = ['Pomme', 'Banane', 'Cerise']`, affiche-les avec un `ListView.builder` (pas de `.map()` direct), chaque élément dans un `ListTile`.

## Exercice 2 — Navigation avec paramètres

Crée deux écrans `UserListScreen` et `UserDetailScreen`. Depuis la liste, navigue vers le détail via `Navigator.push` en passant un `userId` en paramètre du constructeur de `UserDetailScreen`.

## Exercice 3 — Future et FutureBuilder

Écris une fonction `Future<String> fetchGreeting()` qui simule un appel réseau (`Future.delayed`) puis retourne `"Bonjour depuis le serveur"`. Affiche le résultat avec un `FutureBuilder`, avec un `CircularProgressIndicator` pendant le chargement.

## Exercice 4 — Formulaire validé

Crée un `Form` avec un `TextFormField` (email) dont le `validator` retourne un message d'erreur si le champ est vide ou ne contient pas de `@`. Ajoute un bouton qui déclenche `formKey.currentState!.validate()` et affiche un `SnackBar` si le formulaire est valide.

## Exercice 5 — État partagé avec Provider

Crée un `CounterModel extends ChangeNotifier` avec un `int count` et une méthode `increment()` qui appelle `notifyListeners()`. Expose-le via `ChangeNotifierProvider` à la racine de l'app, et consomme-le dans deux widgets différents (un qui affiche la valeur, un qui l'incrémente) sans passer l'état manuellement en paramètre.
