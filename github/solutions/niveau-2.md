# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

## Exercice 2

```yaml
      - name: Call external API
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: ./scripts/call-api.sh
```

Le secret `API_KEY` doit être préalablement défini dans les paramètres du dépôt (Settings → Secrets and variables → Actions) ; il n'apparaît jamais en clair dans le fichier YAML ni dans les logs (GitHub masque automatiquement sa valeur si elle est imprimée).

## Exercice 3

```
# .github/CODEOWNERS
src/api/       @alice
src/frontend/  @bob
```

## Exercice 4

```markdown
## Description du changement

<!-- Que fait cette PR et pourquoi ? -->

## Comment tester

<!-- Étapes pour vérifier le changement en local -->

## Checklist

- [ ] J'ai ajouté des tests
- [ ] J'ai mis à jour la documentation
```

## Exercice 5

*Squash and merge* compresse tous les commits de la branche en un seul commit sur `main` ; *rebase and merge* rejoue chaque commit de la branche individuellement sur `main`, sans commit de merge, mais en conservant l'historique détaillé de la branche. Pour un projet où chaque commit sur `main` doit correspondre exactement à une PR, **squash and merge** est le bon choix : il garantit mécaniquement cette correspondance 1:1, quel que soit le nombre de commits intermédiaires (fixups, "wip") produits pendant le développement de la branche.
