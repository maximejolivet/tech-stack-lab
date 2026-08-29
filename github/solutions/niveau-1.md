# Solutions — Niveau 1 (Bases)

## Exercice 1

Forker crée une copie indépendante du dépôt sous son propre compte GitHub (sur la plateforme) ; cloner télécharge une copie locale d'un dépôt (le sien ou un fork) pour y travailler. Exemple typique : contribuer à un projet open source où l'on n'a pas les droits d'écriture — on fork le dépôt d'origine, puis on clone son fork en local pour développer.

## Exercice 2

1. Forker le dépôt d'origine sur GitHub.
2. Cloner son fork en local (`git clone`).
3. Créer une branche dédiée à la correction.
4. Commiter les changements et pousser la branche vers son fork.
5. Ouvrir une Pull Request depuis sa branche vers la branche `main` du dépôt d'origine.
6. Répondre aux éventuels commentaires de review jusqu'à approbation et fusion.

## Exercice 3

```
Closes #27
```

## Exercice 4

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
```

## Exercice 5

- **Exiger que les status checks passent avant fusion** : évite qu'une PR contenant des tests en échec (ou même une régression non testée) soit mergée sur `main`, ce qui casserait le déploiement ou la branche pour toute l'équipe.
- **Interdire le push direct sur `main`** : force tout changement à passer par une Pull Request review, évitant qu'un commit non relu (volontaire ou accidentel, y compris un force-push qui réécrirait l'historique) atteigne la branche de production sans contrôle.
