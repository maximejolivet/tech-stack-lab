# Niveau 3 — Avancé

## Exercice 3.1 — Records et sealed classes

Modélise une hiérarchie `sealed interface PaymentEvent permits PaymentSucceeded, PaymentFailed` où `PaymentSucceeded(String orderId, double amount)` et `PaymentFailed(String orderId, String reason)` sont des `record`. Écris une méthode qui traite un `PaymentEvent` avec un `switch` **exhaustif** (sans `default`) et laisse le compilateur garantir la couverture de tous les cas.

## Exercice 3.2 — Concurrence avec ExecutorService

Simule le traitement de 10 "tâches" (méthodes qui font un `Thread.sleep` puis retournent un résultat) en parallèle avec un `ExecutorService` à 4 threads. Récupère tous les résultats (via `Future`) et affiche le temps total d'exécution. Compare (en commentaire) avec le temps qu'aurait pris un traitement séquentiel.

## Exercice 3.3 — Gestion mémoire et fuite simulée

Écris un exemple minimal (en commentaire ou en code, au choix) illustrant comment un `static List` qui accumule des objets sans jamais les retirer peut provoquer une fuite mémoire dans une application Java long-running — contrairement à PHP où chaque requête repart de zéro. Explique en 3-4 lignes pourquoi ce risque est spécifique aux runtimes à état persistant (Java long-running, Node.js, FrankenPHP en mode worker).

## Exercice 3.4 — Design orienté interfaces

Refactore un petit bout de code qui dépend directement d'une classe concrète `MySQLRepository` pour qu'il dépende d'une interface `UserRepository`, avec `MySQLRepository` et une implémentation `InMemoryUserRepository` (pour les tests) qui l'implémentent toutes les deux. Explique en une phrase le lien avec le principe SOLID "D" (Dependency Inversion) — voir [`../../design-patterns/`](../../design-patterns/) pour approfondir.
