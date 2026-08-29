# CI/CD

## 1. Introduction

CI/CD désigne l'automatisation du cycle build → test → déploiement d'une application. Ce dossier s'appuie sur les briques déjà couvertes : [`../docker/`](../docker/) (packager l'application), [`../kubernetes/`](../kubernetes/) (l'orchestrer en production), et [`../github/`](../github/) (GitHub Actions comme moteur d'exécution des pipelines dans les exemples concrets ci-dessous).

**À quoi sert-il ?**
- Détecter les régressions au plus tôt en exécutant automatiquement la suite de tests à chaque changement (Intégration Continue).
- Rendre le déploiement reproductible, rapide et sans intervention manuelle risquée (Déploiement/Livraison Continue).
- Réduire le temps entre "le code est écrit" et "le code est en production" en toute confiance.

**CI vs CD — la nuance qui compte** :
- **Continuous Integration (CI)** : chaque changement est automatiquement construit et testé, intégré fréquemment à la branche principale pour éviter les divergences longues entre branches.
- **Continuous Delivery** : le code est automatiquement préparé pour être déployé en production à tout moment (artefact prêt, testé), mais le déclenchement du déploiement reste une décision manuelle.
- **Continuous Deployment** : chaque changement qui passe la pipeline est déployé en production **automatiquement**, sans intervention humaine.

**Avantages** : détection précoce des bugs, cadence de livraison plus rapide et plus sûre, moins de "je te jure que ça marchait en local".
**Limites** : une pipeline mal conçue (tests lents, flaky, pas de rollback) ralentit l'équipe autant qu'elle est censée l'accélérer ; nécessite une discipline de tests réelle en amont (voir [`../testing/`](../testing/)) — l'automatisation ne compense pas l'absence de tests.

## 2. Prérequis

- Notions Git/GitHub : branches, Pull Requests (voir [`../git/`](../git/), [`../github/`](../github/)).
- Bases Docker pour comprendre le packaging d'un artefact déployable (voir [`../docker/`](../docker/)).
- Notions de tests automatisés (voir [`../testing/`](../testing/)) — une pipeline CI/CD n'a de valeur que si elle exécute de vrais tests.

## 3. Rappel des bases 🟢

### 01 - Les étapes d'une pipeline

**Explication** — Une pipeline typique enchaîne : **build** (compiler/packager le code), **test** (unitaires, puis intégration), **package** (produire un artefact versionné, ex. image Docker), **deploy** (livrer l'artefact sur un environnement).

```yaml
# .github/workflows/pipeline.yml
name: Pipeline
on: [push]

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

**Bonne pratique** : faire échouer la pipeline le plus tôt possible (fail fast) — un lint qui échoue en 10 secondes doit bloquer avant de lancer une suite de tests d'intégration qui prend 10 minutes.

### 02 - Environnements

**Explication** — Une pipeline déploie généralement vers plusieurs environnements successifs : **dev/staging** (validation continue, données de test) puis **production** (utilisateurs réels), parfois avec un environnement de **review** éphémère par Pull Request.

**Bonne pratique** : staging doit répliquer la production le plus fidèlement possible (mêmes versions, config proche) — un bug qui n'apparaît qu'en prod signale un écart d'environnement à corriger.

### 03 - Artefacts et versioning

**Explication** — Un artefact (image Docker, binaire, bundle) construit une fois par la pipeline doit être **le même** qui traverse tous les environnements jusqu'en production — jamais reconstruit à chaque étape (ce qui réintroduirait un risque de divergence).

```yaml
      - run: docker build -t mon-app:${{ github.sha }} .
      - run: docker push registry.example.com/mon-app:${{ github.sha }}
```

**Erreur fréquente** : rebuilder l'image à l'étape de déploiement au lieu de réutiliser celle testée à l'étape précédente — on perd la garantie que "ce qui a été testé est ce qui est déployé".

### 04 - Cache de build

**Explication** — Mettre en cache les dépendances (node_modules, vendor, layers Docker) entre les exécutions accélère drastiquement la pipeline.

```yaml
      - uses: actions/cache@v4
        with:
          path: ~/.npm
          key: npm-${{ hashFiles('package-lock.json') }}
```

**Bonne pratique** : la clé de cache doit inclure un hash du fichier de lock (`package-lock.json`, `composer.lock`) — le cache s'invalide automatiquement dès que les dépendances changent, sans nécessiter de purge manuelle.

### 05 - Déploiement automatisé (exemple simple)

**Explication** — Un job de déploiement, déclenché uniquement sur la branche principale, qui pousse l'artefact validé vers l'environnement cible.

```yaml
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

**Bonne pratique** : conditionner le déploiement sur le succès explicite de l'étape de test (`needs: test`) — jamais de déploiement en parallèle ou indépendant des tests.

## 4. Concepts intermédiaires 🟡

- **Stratégies de déploiement** :
  - **Rolling update** : remplace progressivement les instances anciennes par les nouvelles, sans interruption de service (défaut de Kubernetes — voir [`../kubernetes/`](../kubernetes/)).
  - **Blue-green** : deux environnements identiques (bleu = actuel, vert = nouvelle version) ; on bascule le trafic d'un coup une fois le vert validé, avec un retour arrière instantané en cas de problème.
  - **Canary** : la nouvelle version reçoit d'abord une petite fraction du trafic réel (ex. 5%), surveillée avant d'augmenter progressivement — limite l'impact d'une régression non détectée en pipeline.
- **Migrations de base de données zero-downtime** : une migration qui ajoute une colonne `NOT NULL` casse l'ancienne version du code encore en cours de déploiement (rolling update) ; la pratique sûre est en plusieurs étapes — colonne nullable d'abord, backfill, puis contrainte appliquée dans un déploiement ultérieur une fois tout le code migré.
- **Feature flags** : découpler le déploiement du code (peut arriver en prod inactif) de son activation (bascule runtime, sans redéploiement) — permet un rollback instantané d'une fonctionnalité sans revert de code.
- **Sécurité de la pipeline** : ne jamais logguer un secret, scanner les dépendances et l'image (SAST/dependency scanning) comme étape de pipeline à part entière — voir [`../security/`](../security/).
- **Notifications et visibilité** : faire remonter l'état de la pipeline (Slack, statut de commit) pour que l'équipe ait un signal immédiat en cas d'échec, plutôt que de le découvrir a posteriori.

## 5. Concepts avancés 🟠🔴

- **Rollback automatique piloté par le monitoring** : une pipeline avancée surveille des métriques post-déploiement (taux d'erreur, latence) sur une fenêtre de temps courte après un déploiement canary/blue-green, et déclenche un rollback automatique si un seuil est dépassé — sans attendre une intervention humaine.
- **Progressive delivery** : généralisation du canary — combiner feature flags + déploiement progressif + métriques automatisées pour livrer en continu avec un risque contrôlé, plutôt que par gros lots espacés.
- **Infrastructure as Code dans la pipeline** : la pipeline ne déploie pas que l'application, mais aussi l'infrastructure qui la porte (Terraform, Helm charts pour Kubernetes), versionnée et review comme du code.
- **Pipeline multi-étapes avec approbations** : un déploiement en production peut nécessiter une approbation manuelle explicite (gate) après un déploiement automatique en staging — combine automatisation et contrôle humain aux points critiques.
- **Idempotence des déploiements** : une pipeline doit pouvoir être rejouée sans effet de bord destructeur si elle échoue à mi-chemin (ex. un déploiement Kubernetes réappliqué ne doit pas dupliquer des ressources).
- **Observabilité de la pipeline elle-même** : suivre la durée, le taux d'échec et le taux de flakiness de la pipeline comme des métriques à part entière — une pipeline qui devient lente ou instable dégrade la vélocité de toute l'équipe, symptôme à traiter comme une dette technique.

## 6. Commandes / syntaxe à connaître

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps: [ ... ]
  test:
    needs: build
    steps: [ ... ]
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    environment: production
    steps: [ ... ]
```

```bash
docker build -t mon-app:$(git rev-parse --short HEAD) .
docker push registry.example.com/mon-app:$(git rev-parse --short HEAD)
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Pipeline complète pour une application conteneurisée**

- Un job `build` qui installe les dépendances et construit l'application.
- Un job `test` (dépendant de `build`) qui lance la suite de tests.
- Un job `package` qui construit une image Docker taguée avec le SHA du commit et la pousse vers un registre.
- Un job `deploy-staging` automatique sur chaque push vers `main`, et un job `deploy-production` nécessitant une approbation manuelle (environment protégé).
- Mise en cache des dépendances pour accélérer les runs successifs.
- Bonus : simuler un déploiement canary (10% du trafic) avec un rollback automatique si un endpoint de health-check répond en erreur.

## Checklist

- [ ] Comprendre les fondamentaux (CI vs Continuous Delivery vs Continuous Deployment, étapes d'une pipeline)
- [ ] Savoir écrire une pipeline avec build/test/deploy enchaînés
- [ ] Maîtriser la syntaxe principale (jobs, `needs`, artefacts, cache)
- [ ] Comprendre les concepts importants (stratégies de déploiement, migrations zero-downtime, feature flags)
- [ ] Savoir debugger une pipeline qui échoue (logs, réexécution d'un job isolé)
- [ ] Connaître les bonnes pratiques (fail fast, artefact unique de bout en bout, sécurité des secrets)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (rollback piloté par monitoring, progressive delivery, IaC dans la pipeline)

## 10. Ressources

- [roadmap.sh — DevOps](https://roadmap.sh/devops) — roadmap role-based qui couvre CI/CD dans son ensemble (build, test, déploiement, monitoring).
- [GitHub Actions — documentation](https://docs.github.com/actions) — moteur de pipeline utilisé dans les exemples de ce dossier (voir [`../github/`](../github/)).
- [The Twelve-Factor App](https://12factor.net/fr/) — principes sous-jacents à un déploiement propre et reproductible (config, build/release/run séparés).
- [Continuous Delivery (Jez Humble & David Farley)](https://continuousdelivery.com/) — référence historique sur les principes du domaine.
