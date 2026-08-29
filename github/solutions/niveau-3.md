# Solutions — Niveau 3 (Avancé)

## Exercice 1

```yaml
# .github/workflows/reusable-ci.yml
name: Reusable CI
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  call-ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '20'
```

## Exercice 2

Une composite action se définit dans un dossier dédié (ex. `.github/actions/setup-project/`) contenant un fichier `action.yml` qui déclare `runs: { using: 'composite', steps: [...] }`, avec la séquence checkout + `setup-node` + `npm ci` en steps internes. Elle s'utilise ensuite depuis n'importe quel workflow du dépôt via `uses: ./.github/actions/setup-project`.

## Exercice 3

L'environment `production` se configure dans Settings → Environments, avec une règle "Required reviewers" listant au moins un utilisateur/équipe habilité à approuver le déploiement.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh
```

Le job `deploy` reste en attente ("Waiting") jusqu'à ce qu'un reviewer autorisé approuve explicitement l'exécution.

## Exercice 4

```yaml
on:
  push:
    paths:
      - 'packages/api/**'
  pull_request:
    paths:
      - 'packages/api/**'
```

## Exercice 5

Un outil de semantic release analyse le message de chaque commit fusionné sur `main` depuis le dernier tag, et en extrait un type structuré (ex. `fix:`, `feat:`, `feat!:` ou `BREAKING CHANGE:` dans le corps) selon la convention Conventional Commits. Un `fix:` déclenche un incrément de version *patch*, un `feat:` un incrément *mineur*, et une rupture de compatibilité (`!` ou `BREAKING CHANGE`) un incrément *majeur*. Cette convention est nécessaire car l'outil n'a aucun moyen d'inférer sémantiquement l'impact d'un changement à partir du code seul — il doit s'appuyer sur une déclaration explicite et structurée fournie par l'auteur du commit, ce qui suppose que toute l'équipe respecte le même format.
