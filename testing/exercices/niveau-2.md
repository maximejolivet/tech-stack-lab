# Exercices Testing — Niveau 2 (Intermédiaire)

## Exercice 1 — Cycle TDD

Décris, étape par étape (red-green-refactor), comment tu développerais en TDD une fonction `isValidEmail(email)` qui doit retourner `true` pour un email au format valide et `false` sinon, en partant d'un seul cas simple avant d'en ajouter d'autres.

## Exercice 2 — Unitaire vs intégration

Pour une fonction `createOrder(cart, paymentService)` qui calcule un total et appelle un service de paiement externe, décris : (a) ce que testerait un test unitaire de cette fonction, et (b) ce qu'apporterait en plus un test d'intégration qui utilise un vrai service de paiement de test (sandbox).

## Exercice 3 — Danger des snapshots

Un test de snapshot capture le HTML rendu d'un composant `ProductCard`. Un développeur change juste une classe CSS sans impact fonctionnel, le test échoue, et l'équipe a pris l'habitude de relancer la commande de mise à jour des snapshots sans relire le diff. Explique en 2-3 phrases le risque à moyen terme de cette habitude.

## Exercice 4 — Couverture de code trompeuse

Voici un test avec 100% de couverture de la fonction testée mais qui ne détecterait aucune régression si la fonction était cassée :

```
function divide(a, b) { return a / b; }

test("divide runs", () => {
  divide(10, 2);
});
```

Explique pourquoi, et corrige le test.

## Exercice 5 — Corriger un test flaky

Un test attend qu'une requête réseau simulée se termine avec `sleep(1000)` avant de vérifier le résultat, et échoue occasionnellement en CI (environnement plus lent). Propose une correction qui élimine la dépendance à un délai fixe.
