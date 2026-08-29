# Testing

## 1. Introduction

Le testing regroupe les pratiques et principes qui garantissent qu'un logiciel se comporte comme attendu, de façon automatisée et reproductible. Ce dossier est volontairement **agnostique du langage/framework** : les outils concrets (PHPUnit/Pest, Jest/Testing Library, PyTest...) sont couverts dans les dossiers de chaque techno ([`../php/`](../php/), [`../laravel/`](../laravel/), [`../react/`](../react/), [`../django/`](../django/)...). Ici, on couvre les **principes transverses** applicables à n'importe quelle stack.

**À quoi sert-il ?**
- Détecter une régression avant qu'un utilisateur (ou la CI/CD, voir [`../ci-cd/`](../ci-cd/)) ne la découvre.
- Documenter le comportement attendu du code de façon exécutable — un test à jour ne ment jamais, contrairement à un commentaire.
- Permettre de refactorer avec confiance : un changement interne qui casse un test révèle immédiatement une régression de comportement.

**Où se situe-t-il dans le cycle de développement ?** Transverse à tout le cycle : écrit avant (TDD), pendant, ou après le code métier, exécuté en local par le développeur et systématiquement en CI avant toute fusion/déploiement.

**Avantages** : confiance pour livrer et refactorer, documentation vivante du comportement, détection précoce qui coûte infiniment moins cher qu'un bug découvert en production.
**Limites** : une suite de tests mal conçue (trop couplée à l'implémentation, lente, flaky) devient un frein plutôt qu'une aide ; 100% de couverture ne garantit pas l'absence de bugs (voir section couverture).

## 2. Prérequis

- Un langage et son framework de test associé maîtrisés au moins aux bases (voir le dossier de la techno concernée).
- Comprendre ce qu'est une fonction pure et un effet de bord — notion centrale pour écrire du code testable.

## 3. Rappel des bases 🟢

### 01 - La pyramide de tests

**Explication** — Modèle classique répartissant l'effort de test en trois niveaux, du plus nombreux/rapide/isolé au moins nombreux/lent/réaliste :
- **Tests unitaires** (base, nombreux) : testent une fonction/classe isolée, sans dépendances externes (DB, réseau).
- **Tests d'intégration** (milieu) : testent l'interaction entre plusieurs composants réels (ex. un contrôleur + une vraie base de données).
- **Tests end-to-end / E2E** (sommet, peu nombreux) : testent l'application entière comme un utilisateur réel (navigateur piloté, appels HTTP réels).

**Bonne pratique** : respecter la forme pyramidale — beaucoup de tests unitaires rapides, peu de tests E2E lents et coûteux à maintenir. Une pyramide inversée ("cône de glace" : surtout des E2E) rend la suite lente et fragile.

### 02 - Anatomie d'un test unitaire (Arrange-Act-Assert)

**Explication** — Structure en trois temps : **Arrange** (préparer les données/état nécessaires), **Act** (exécuter l'action testée), **Assert** (vérifier le résultat).

```
// Arrange
const cart = new Cart();
cart.add({ id: 1, price: 10 });

// Act
const total = cart.getTotal();

// Assert
expect(total).toBe(10);
```

**Erreur fréquente** : mélanger plusieurs assertions non liées dans un seul test ("test fourre-tout") — quand il échoue, on ne sait pas immédiatement laquelle a échoué ni pourquoi. Un test = un comportement vérifié.

### 03 - Qu'est-ce qu'un bon test ?

**Explication** — Un bon test est **isolé** (n'affecte pas et n'est pas affecté par d'autres tests, peut tourner dans n'importe quel ordre), **répétable** (même résultat à chaque exécution, aucune dépendance au hasard/à l'heure système non maîtrisée), **rapide** (surtout pour la base de la pyramide), et **lisible** (le nom du test décrit le comportement attendu, pas l'implémentation).

**Bonne pratique** : nommer un test par son comportement attendu (`calculates total with multiple items`) plutôt que par la méthode testée (`test_getTotal`) — le nom doit rester compréhensible même sans lire le code.

### 04 - Test doubles : mock, stub, spy, fake

**Explication** — Vocabulaire pour désigner un objet qui remplace une vraie dépendance pendant un test :
- **Stub** : renvoie une réponse préprogrammée fixe, sans vérifier comment il a été appelé.
- **Mock** : comme un stub, mais on **vérifie** en plus qu'il a été appelé avec certains arguments / un certain nombre de fois.
- **Spy** : enveloppe un objet réel tout en enregistrant les appels qu'il reçoit, pour les vérifier ensuite.
- **Fake** : une implémentation simplifiée mais fonctionnelle (ex. une base de données en mémoire à la place d'une vraie base).

**Erreur fréquente** : mocker une dépendance qu'on ne possède/contrôle pas soi-même (ex. une librairie tierce complexe) de façon trop détaillée — le mock devient une réplique fragile de l'implémentation interne de la dépendance, qui casse à chaque mise à jour de celle-ci sans qu'un vrai bug existe.

### 05 - Assertions et matchers

**Explication** — La plupart des frameworks de test fournissent un vocabulaire d'assertions expressif (`toEqual`, `toContain`, `toThrow`...) plutôt qu'un simple `assert(a === b)`.

**Bonne pratique** : utiliser le matcher le plus spécifique disponible (`toHaveLength(3)` plutôt que `expect(arr.length).toBe(3)`) — un message d'échec plus précis accélère le diagnostic.

### 06 - Setup et teardown

**Explication** — Hooks exécutés avant/après chaque test (ou chaque suite) pour préparer et nettoyer l'état partagé (connexion DB de test, fixtures).

**Bonne pratique** : chaque test doit repartir d'un état propre et connu — ne jamais dépendre de l'état laissé par un test précédent, même s'ils s'exécutent dans le même fichier.

## 4. Concepts intermédiaires 🟡

- **TDD (Test-Driven Development) — cycle red-green-refactor** : écrire d'abord un test qui échoue (*red*, car le comportement n'existe pas encore), écrire le minimum de code pour le faire passer (*green*), puis nettoyer le code sans changer le comportement (*refactor*), tests toujours au vert. Discipline plutôt qu'obligation — utile pour clarifier la spécification avant l'implémentation.
- **Tests d'intégration ciblés** : tester la vraie interaction entre deux composants (ex. un repository et une vraie base de données de test) plutôt que de tout mocker — détecte des bugs qu'aucun test unitaire isolé ne peut voir (mauvais mapping SQL, contrainte de schéma).
- **Test de snapshot** : capturer la sortie d'un composant/fonction (ex. HTML rendu) et la comparer à une version de référence enregistrée. Rapide à écrire mais fragile : un changement volontaire mineur casse le snapshot sans qu'un vrai bug existe, et l'équipe prend l'habitude de "re-générer" le snapshot sans le relire — à réserver à des sorties stables et à toujours relire le diff avant de le mettre à jour.
- **Couverture de code (code coverage)** : pourcentage de lignes/branches exécutées par la suite de tests. Utile comme signal d'alerte (0% de couverture sur un module critique est un vrai problème) mais **n'est pas une mesure de qualité** — un test peut exécuter une ligne sans vérifier réellement son résultat (assertion absente ou trop faible), gonflant la couverture sans réduire le risque de bug.
- **Flaky tests — causes et corrections** : un test qui échoue de façon intermittente sans changement de code sape la confiance dans toute la suite. Causes fréquentes : dépendance à un ordre d'exécution, `sleep`/timing non déterministe au lieu d'attendre une condition, dépendance réseau non isolée, horloge système non contrôlée (`Date.now()`). Corriger la cause plutôt que de relancer le test en boucle jusqu'à ce qu'il passe.
- **Fixtures et factories** : générer des données de test réalistes et variées par du code plutôt que des données codées en dur répétées dans chaque test — limite la duplication et documente les invariants du modèle de données.

## 5. Concepts avancés 🟠🔴

- **Mutation testing** : au lieu de mesurer si le code est *exécuté* par les tests (couverture classique), un outil de mutation testing modifie volontairement le code source (ex. `>` devient `>=`) et vérifie qu'au moins un test échoue suite à cette "mutation". Un mutant qui survit (aucun test ne le détecte) révèle une assertion manquante là où la couverture de lignes semblait pourtant à 100%.
- **Contract testing** : dans une architecture à plusieurs services, tester non pas l'intégration réelle bout-en-bout (lente, fragile) mais un **contrat** formel entre un consommateur et un fournisseur d'API (ex. Pact) — chaque côté vérifie indépendamment qu'il respecte le contrat partagé, sans avoir besoin de faire tourner l'autre service pendant le test.
- **Property-based testing** : au lieu d'écrire des exemples fixes (`assert add(2, 3) == 5`), on décrit une **propriété** générale que la fonction doit toujours respecter (`add(a, b) == add(b, a)` pour tout `a, b`), et un outil (ex. Hypothesis en Python, fast-check en JS) génère automatiquement de nombreux cas, y compris des cas limites auxquels on n'aurait pas pensé.
- **Tests de charge et de performance** : vérifier que le système respecte des contraintes de latence/débit sous charge réaliste ou extrême (voir [`../system-design/`](../system-design/) pour la conception associée) — catégorie distincte des tests fonctionnels, souvent exécutée séparément de la pipeline CI standard.
- **Test en production (canary, feature flags)** : certaines validations (comportement sous trafic réel, performance réelle) ne sont fiables qu'en observant un déploiement progressif en production plutôt qu'un environnement de test simulé — voir [`../ci-cd/`](../ci-cd/) pour les stratégies de déploiement associées.
- **Culture de test dans une équipe** : la valeur d'une suite de tests dépend autant de sa conception que de la discipline collective à la maintenir (ne pas désactiver un test qui gêne, corriger un flaky test immédiatement plutôt que de le laisser s'accumuler) — un facteur souvent plus déterminant que le choix d'un outil particulier.

## 6. Commandes / syntaxe à connaître

```
// Structure générique d'un test (pseudo-code, la syntaxe exacte dépend du framework)
describe("Cart", () => {
  it("calculates total with multiple items", () => {
    // Arrange
    const cart = new Cart();
    cart.add({ price: 10 });
    cart.add({ price: 5 });

    // Act
    const total = cart.getTotal();

    // Assert
    expect(total).toBe(15);
  });
});
```

```
// Vocabulaire des test doubles
stub  → réponse fixe préprogrammée
mock  → réponse programmée + vérification des appels reçus
spy   → objet réel instrumenté pour observer les appels
fake  → implémentation simplifiée mais fonctionnelle
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Concevoir la stratégie de test d'un module "Panier d'achat"**

Sans nécessairement écrire de code dans un langage précis (ou en pseudo-code/langage de ton choix), pour un module `Cart` (ajout/suppression d'article, calcul du total avec remises, vérification du stock via un service externe) :
- Lister les cas de test unitaires à couvrir pour la logique de calcul du total (cas nominal, panier vide, quantité négative, remise qui dépasse le total).
- Identifier quelle dépendance du module doit être remplacée par un test double pour les tests unitaires, et lequel (stub/mock/fake) est le plus adapté et pourquoi.
- Décrire un test d'intégration qui vérifierait le comportement réel avec le service de stock (sans le mocker).
- Repérer un cas qui se prêterait bien au property-based testing (quelle propriété générale énoncer ?).
- Bonus : identifier une source probable de flakiness si ce module dépendait d'une horloge système pour appliquer une remise "valable aujourd'hui", et proposer une correction.

## Checklist

- [ ] Comprendre les fondamentaux (pyramide de tests, Arrange-Act-Assert, ce qui fait un bon test)
- [ ] Savoir écrire un test unitaire isolé et lisible
- [ ] Maîtriser le vocabulaire des test doubles (mock, stub, spy, fake) et savoir quand utiliser lequel
- [ ] Comprendre les concepts importants (TDD, couverture de code et ses limites, tests flaky)
- [ ] Savoir diagnostiquer et corriger un test flaky
- [ ] Connaître les bonnes pratiques (un test = un comportement, isolation, nommage par comportement)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (mutation testing, contract testing, property-based testing)

## 10. Ressources

- [Martin Fowler — Test doubles](https://martinfowler.com/bliki/TestDouble.html) — référence sur le vocabulaire mock/stub/spy/fake.
- [Martin Fowler — Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) — approfondissement de la pyramide de tests.
- [Hypothesis (property-based testing, Python)](https://hypothesis.readthedocs.io/) et [fast-check (JS/TS)](https://fast-check.dev/) — outils de référence.
- [Pact](https://docs.pact.io/) — outil de référence pour le contract testing entre services.
- Chaque techno de ce repo documente ses outils de test spécifiques dans son propre dossier (ex. Testing Library en [`../react/`](../react/), Pest/PHPUnit en [`../laravel/`](../laravel/)) ; il n'existe pas de roadmap.sh dédiée au testing en tant que discipline transverse, voir [roadmap.sh — QA](https://roadmap.sh/qa) pour un angle plus proche du métier QA.
