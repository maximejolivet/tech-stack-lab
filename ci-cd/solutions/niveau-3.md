# Solutions — Niveau 3 (Avancé)

## Exercice 1

1. Déployer la nouvelle version uniquement sur 10% des instances (ou router 10% du trafic vers elle).
2. Pendant une fenêtre d'observation de 5 minutes, interroger en continu le système de monitoring pour le taux d'erreur des requêtes servies par cette version canary.
3. Si le taux d'erreur reste sous 2% à la fin de la fenêtre, augmenter progressivement le trafic vers la nouvelle version jusqu'à 100%.
4. Si le taux d'erreur dépasse 2% à un moment de la fenêtre, arrêter immédiatement l'augmentation de trafic, rebasculer 100% du trafic vers l'ancienne version, et faire échouer la pipeline pour notifier l'équipe — sans attendre la fin de la fenêtre d'observation.

## Exercice 2

```yaml
  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh production
```

Le job `deploy-production` reste en attente d'une approbation manuelle définie sur l'environment `production` (Settings → Environments → Required reviewers) avant de s'exécuter.

## Exercice 3

Une pipeline de déploiement idempotente peut être rejouée plusieurs fois de suite, y compris après un échec partiel, sans produire d'effet différent ou destructeur par rapport à une exécution unique réussie. Exemple de pipeline NON idempotente : un script qui exécute `INSERT INTO audit_log VALUES (...)` à chaque déploiement sans vérifier si l'entrée existe déjà — rejouer la pipeline après un échec en cours de route dupliquerait les lignes d'audit, ou pire, un script qui crée une nouvelle ressource cloud (`create-database`) sans vérifier son existence préalable échouerait ou créerait une ressource en double selon l'implémentation.

## Exercice 4

Le changement d'infrastructure (Terraform) doit être appliqué **avant** le déploiement du code applicatif qui en dépend, dans une étape dédiée de la pipeline, et non en parallèle ni après. Si le code est déployé avant que l'infrastructure ne soit prête (ex. une nouvelle table ou un nouveau paramètre de connexion à une base managée), il tentera d'utiliser une ressource qui n'existe pas encore et échouera au démarrage ; l'ordre inverse garantit que l'infrastructure cible est disponible au moment où le nouveau code applicatif en a besoin.

## Exercice 5

- **Durée moyenne d'exécution de la pipeline** : une pipeline qui s'allonge progressivement indique une dette (tests lents non parallélisés, cache mal configuré) qui ralentit chaque itération de l'équipe, même sans échec.
- **Taux d'échec (flakiness)** : un taux d'échec élevé sans lien avec de vrais bugs (tests instables, dépendances réseau non mockées) érode la confiance de l'équipe dans la pipeline — les échecs finissent par être ignorés ("relance et ça passera"), ce qui masque les vrais échecs.
- **Fréquence des rollbacks post-déploiement** : un taux de rollback élevé indique que la pipeline valide mal les changements avant production (couverture de tests insuffisante, environnement de staging trop différent de la prod), signal qu'il faut renforcer les étapes de vérification en amont plutôt que de compter sur le rollback comme filet de sécurité.
