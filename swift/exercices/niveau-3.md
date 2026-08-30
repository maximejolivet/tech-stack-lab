# Niveau 3 — Avancé

## Exercice 3.1 — Protocol-oriented programming

Définis un `protocol Payable { var amount: Double { get }; func describe() -> String }` avec une **extension** fournissant une implémentation par défaut de `describe()` (ex. `"Montant : \(amount)€"`). Crée deux `struct` conformes (`Invoice`, `Refund`) qui n'implémentent pas `describe()` elles-mêmes, et vérifie qu'elles héritent bien du comportement par défaut.

## Exercice 3.2 — Cycle de rétention et ARC

Modélise deux classes `Owner` (avec `var pet: Pet?`) et `Pet` (avec `var owner: Owner?`), sans `weak`. Explique en 2-3 lignes pourquoi ces deux instances qui se référencent mutuellement ne seraient jamais libérées de la mémoire (cycle de rétention). Corrige en ajoutant `weak` au bon endroit et explique pourquoi ce sens précis (et pas l'autre) est le bon choix.

## Exercice 3.3 — Actor et concurrence

Écris un `actor Counter` avec une propriété privée `count` et des méthodes `increment()` et `getValue() -> Int`. Depuis plusieurs `Task` lancées en parallèle appelant `increment()`, vérifie qu'aucune valeur n'est perdue (contrairement à ce qui se passerait avec une simple `class` mutée depuis plusieurs threads sans synchronisation). Explique en une phrase ce que le compilateur garantit ici que Kotlin n'impose pas par défaut (voir [`../../kotlin/`](../../kotlin/), section coroutines).

## Exercice 3.4 — Design orienté protocoles (comparaison avec Kotlin)

Reprends l'exercice 3.4 du dossier Kotlin ([`../../kotlin/exercices/niveau-3.md`](../../kotlin/exercices/niveau-3.md)) : un `UserService` qui dépend directement d'un `MySQLUserRepository`. Refactore-le en Swift pour dépendre d'un `protocol UserRepository`, avec `MySQLUserRepository` et `InMemoryUserRepository` qui s'y conforment toutes les deux. Compare en 2-3 lignes la syntaxe des protocoles Swift avec les interfaces Kotlin sur ce même exemple.
