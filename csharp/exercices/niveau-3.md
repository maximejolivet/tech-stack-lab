# Niveau 3 — Avancé

## Exercice 3.1 — Records et pattern matching exhaustif

Modélise deux `record` : `PaymentSucceeded(string OrderId, double Amount)` et `PaymentFailed(string OrderId, string Reason)`, tous deux héritant d'une classe abstraite (ou d'une interface) commune `PaymentEvent`. Écris une méthode `string Describe(PaymentEvent evt)` avec un `switch` expression basé sur le pattern matching de type (`PaymentSucceeded s => ...`) qui couvre tous les cas.

## Exercice 3.2 — Concurrence avec async/await et Task.WhenAll

Simule le traitement de 10 "tâches" (méthodes `async Task<int>` qui font un `await Task.Delay(500)` puis retournent un résultat) en parallèle avec `Task.WhenAll`. Affiche le temps total d'exécution. Compare (en commentaire) avec le temps qu'aurait pris un traitement séquentiel (`await` un par un dans une boucle).

## Exercice 3.3 — Générations du garbage collector

Écris un exemple minimal illustrant comment une `static List<byte[]>` qui accumule des tableaux sans jamais les retirer peut provoquer une fuite mémoire dans une application .NET long-running. Explique en 3-4 lignes le rôle des générations Gen0/Gen1/Gen2 du GC dans ce scénario : pourquoi ces objets, jamais libérés, finissent par être promus en Gen2 et pourquoi cela aggrave le coût des collections suivantes.

## Exercice 3.4 — Design orienté interfaces

Refactore un petit bout de code qui dépend directement d'une classe concrète `SqlUserRepository` pour qu'il dépende d'une interface `IUserRepository`, avec `SqlUserRepository` et une implémentation `InMemoryUserRepository` (pour les tests) qui l'implémentent toutes les deux. Explique en une phrase le lien avec le principe SOLID "D" (Dependency Inversion) — voir [`../../design-patterns/`](../../design-patterns/) pour approfondir.
