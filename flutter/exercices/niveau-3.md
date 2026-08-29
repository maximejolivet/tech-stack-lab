# Exercices Flutter — Niveau 3 (Avancé)

## Exercice 1 — Widgets const et performance

Prends un widget parent qui reconstruit fréquemment (ex: via un `Timer` qui appelle `setState` chaque seconde) et contient un widget enfant statique (`Text('Titre fixe')`). Marque l'enfant `const`, puis explique en 3-4 lignes pourquoi Flutter peut éviter de le reconstruire malgré les `setState` répétés du parent.

## Exercice 2 — AOT vs JIT (conceptuel)

Sans écrire de code, explique en 4-5 lignes la différence entre le mode debug (JIT, hot reload actif) et le mode release (`flutter build`, AOT) — pourquoi le hot reload n'est-il disponible qu'en JIT, et pourquoi une app en AOT démarre-t-elle généralement plus vite ?

## Exercice 3 — Riverpod vs Provider (conceptuel)

Réimplémente le `CounterModel` de l'exercice 2.5 avec Riverpod (`StateNotifierProvider` ou `NotifierProvider` selon la version) au lieu de `Provider`/`ChangeNotifier`. Explique en 2-3 lignes le principal avantage pratique de Riverpod par rapport à Provider (indice : dépendance au `BuildContext`).

## Exercice 4 — Platform channel (conceptuel)

Sans écrire de code Swift/Kotlin, décris en 4-5 lignes les étapes nécessaires pour exposer une fonctionnalité native (ex: lire le niveau de batterie de l'appareil) à Dart via un `MethodChannel` — quel code s'exécute côté Dart, quel code s'exécute côté natif, et comment communiquent-ils.
