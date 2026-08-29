# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

Le blue-green bascule tout le trafic d'un coup d'un environnement à l'autre (rollback instantané en rebasculant), tandis que le canary n'expose la nouvelle version qu'à une fraction du trafic réel, augmentée progressivement après observation. Le canary est préférable dès qu'il y a un trafic suffisant pour qu'un petit pourcentage reste statistiquement significatif (détecter une régression sur 5% d'utilisateurs réels) ; sur une application à très faible trafic, un canary n'aurait pas assez de volume pour être représentatif, et le blue-green (bascule complète avec rollback rapide) est plus adapté.

## Exercice 2

1. **Déploiement 1** : ajouter la colonne `email_verified` en `NULLABLE` (pas encore `NOT NULL`) — les anciennes instances qui ignorent cette colonne continuent de fonctionner normalement.
2. **Backfill** : exécuter un script qui remplit `email_verified` pour toutes les lignes existantes (ex. `false` par défaut), une fois que le code qui écrit correctement cette colonne est déployé partout.
3. **Déploiement 2**, une fois que **toutes** les instances (anciennes incluses) sont remplacées par la version qui connaît cette colonne : appliquer la contrainte `NOT NULL` définitive.

## Exercice 3

Un feature flag permet de désactiver une fonctionnalité à l'exécution (bascule d'une valeur de configuration, souvent instantanée) sans toucher au code déployé ni redéployer — le "rollback" devient une simple bascule de flag plutôt qu'un retour en arrière du déploiement. Cette approche ne suffit pas quand le bug affecte du code qui s'exécute même flag désactivé (ex. une erreur dans le chemin d'initialisation, une corruption de données déjà écrite) : dans ce cas, seul un vrai rollback de code (ou une correction des données) résout le problème.

## Exercice 4

```yaml
  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy-staging.sh
```

## Exercice 5

```yaml
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  deploy:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```
