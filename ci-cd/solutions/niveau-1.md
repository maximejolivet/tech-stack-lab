# Solutions — Niveau 1 (Bases)

## Exercice 1

- **CI (Intégration Continue)** : chaque changement est automatiquement construit et testé dès qu'il est poussé, pour détecter les régressions au plus tôt.
- **Continuous Delivery** : le code passé en CI est automatiquement préparé pour être déployé en production à tout moment, mais le déclenchement réel du déploiement reste une décision manuelle.
- **Continuous Deployment** : chaque changement qui passe la CI est déployé en production automatiquement, sans validation humaine.

Exemple : deux équipes ont exactement la même pipeline de tests/build. L'une (Continuous Delivery) clique sur un bouton "Déployer" une fois par jour après relecture du changelog ; l'autre (Continuous Deployment) voit chaque merge sur `main` arriver en production en quelques minutes, sans étape manuelle.

## Exercice 2

```yaml
name: Pipeline
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

## Exercice 3

Le principe de fail fast consiste à exécuter d'abord les vérifications les moins coûteuses en temps : si le lint échoue, la pipeline s'arrête en 10 secondes au lieu d'attendre 5 minutes de tests d'intégration pour un problème qui aurait pu être détecté immédiatement. Dans l'ordre inverse, un développeur découvrirait une simple erreur de style seulement après avoir attendu la pipeline la plus longue, ce qui gaspille du temps de feedback à chaque itération.

## Exercice 4

```yaml
      - uses: actions/cache@v4
        with:
          path: ~/.npm
          key: npm-${{ hashFiles('package-lock.json') }}
      - run: npm ci
```

## Exercice 5

Si la pipeline reconstruit l'image à l'étape de déploiement, rien ne garantit que cette nouvelle image contient exactement le code qui a été testé à l'étape précédente (une dépendance pourrait avoir changé de version entre les deux builds, un flag de build différer). On perd la garantie fondamentale de la CI/CD : "ce qui a été validé est ce qui part en production" — un artefact unique, construit une seule fois puis promu d'environnement en environnement, élimine ce risque.
