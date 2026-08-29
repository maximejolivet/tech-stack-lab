# Solutions — Niveau 1 (Bases)

## Exercice 1

Une pyramide inversée rend la suite globalement lente (les E2E sont coûteux en temps d'exécution) et fragile (un test E2E dépend de nombreuses couches — UI, réseau, base — et échoue souvent pour des raisons sans rapport avec un vrai bug). Le feedback devient lent et peu fiable, alors qu'une majorité de tests unitaires rapides et isolés aurait détecté les mêmes régressions en une fraction du temps et avec beaucoup moins de faux positifs.

## Exercice 2

```
test("stack works", () => {
  // Arrange
  const s = new Stack();

  // Act & Assert (état initial)
  expect(s.isEmpty()).toBe(true);

  // Act
  s.push(1);
  s.push(2);

  // Assert
  expect(s.pop()).toBe(2);
});
```

## Exercice 3

`calculates total with multiple items` est le meilleur nom : il décrit le **comportement** attendu en langage naturel, compréhensible même sans lire le code du test. `test_getTotal` nomme la méthode testée plutôt que le comportement (peu informatif si plusieurs tests testent la même méthode dans des cas différents), et `test1` ne donne aucune information exploitable en cas d'échec.

## Exercice 4

C'est un **spy** : il enveloppe/remplace le service réel tout en enregistrant les appels reçus (destinataire, sujet), et le test vérifie ensuite ces appels a posteriori — comportement typique d'un spy. (Un mock aurait été configuré à l'avance avec une attente explicite avant l'exécution ; la distinction mock/spy est parfois utilisée de façon interchangeable selon les frameworks, mais le point clé ici est la vérification des appels reçus, pas une simple réponse préprogrammée comme un stub.)

## Exercice 5

La cause la plus probable est un **état partagé non réinitialisé** entre les deux tests (une variable globale, une base de données de test, un mock non reset) — le premier test modifie un état que le second lit implicitement. La correction consiste à ajouter un `setup`/`teardown` qui réinitialise cet état avant (ou après) chaque test, pour que chaque test parte d'un état connu et indépendant de l'ordre d'exécution.
