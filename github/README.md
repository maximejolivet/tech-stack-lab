# GitHub

## 1. Introduction

GitHub est une plateforme d'hébergement de dépôts Git qui ajoute des outils de collaboration par-dessus le protocole Git (voir [`../git/`](../git/)) : Pull Requests, Issues, Actions (CI/CD), Projects (gestion de travail). Ce dossier suppose Git lui-même acquis — pas de rappel ici sur les commits, branches, merge/rebase, qui sont couverts dans [`../git/`](../git/). On se concentre sur ce qui est **spécifique à GitHub en tant que plateforme**.

**À quoi sert-il ?**
- Héberger et partager du code, en public ou en privé.
- Structurer la collaboration à plusieurs via un flux de review asynchrone (Pull Requests).
- Automatiser build/test/déploiement (GitHub Actions).
- Suivre le travail et la roadmap d'un projet (Issues, Projects).

**Où se situe-t-il dans une architecture de travail ?** Couche collaboration/plateforme au-dessus du protocole Git — un remote parmi d'autres (GitLab, Bitbucket), mais devenu le standard de facto de l'écosystème open source et de la plupart des équipes.

**Avantages** : écosystème d'intégrations et de marketplace Actions immense, gratuit pour l'open source et les petites équipes, CLI (`gh`) et API très complètes.
**Limites** : certaines fonctionnalités de collaboration avancée (environments protégés, required reviewers) réservées aux plans payants sur dépôts privés ; syntaxe Actions propriétaire (portage vers un autre CI nécessite une réécriture).

## 2. Prérequis

- Git solide : commits, branches, merge, rebase, remotes (voir [`../git/`](../git/)).
- Un compte GitHub et une clé SSH configurée pour l'authentification.

## 3. Rappel des bases 🟢

### 01 - Repository, fork et clone

**Explication** — Un **repository** (dépôt) héberge le projet sur GitHub. Un **fork** crée une copie du dépôt sous votre propre compte (utilisé pour contribuer à un projet dont on n'est pas collaborateur direct). Un **clone** copie le dépôt (le vôtre ou un fork) en local pour y travailler.

**Bonne pratique** : fork + Pull Request pour contribuer à un projet externe ; branche directe sur le dépôt d'origine quand on est déjà collaborateur — le fork n'est nécessaire que sans droit d'écriture direct.

### 02 - Pull Requests

**Explication** — Une Pull Request (PR) propose de fusionner une branche dans une autre, avec une phase de review avant fusion. C'est le mécanisme central de collaboration sur GitHub.

```bash
gh pr create --base main --head ma-feature --title "Ajoute la pagination" --body "Description du changement"
```

**Erreur fréquente** : ouvrir une PR trop large (plusieurs fonctionnalités mélangées) — elle devient difficile et lente à review. Garder des PR petites, focalisées sur un seul changement logique.

**Bonne pratique** : lier la PR à l'issue qu'elle résout via un mot-clé dans la description (`Closes #12`) — l'issue se ferme automatiquement à la fusion.

### 03 - Issues

**Explication** — Une Issue trace un bug, une demande de fonctionnalité, ou une tâche. Peut recevoir des labels, un assigné, un milestone, et être liée à une ou plusieurs PR.

**Bonne pratique** : un titre d'issue actionnable et descriptif (`Le formulaire de login rejette les emails avec un +`) plutôt que vague (`Bug login`) — facilite le tri et la recherche plus tard.

### 04 - Code review

**Explication** — Sur une PR, chaque relecteur peut commenter une ligne précise, proposer une modification via une **suggestion** (appliquable en un clic), et rendre un verdict global : *Approve*, *Request changes*, ou *Comment*.

**Bonne pratique** : une review porte sur le code, pas sur la personne — formuler les remarques en questions ou en suggestions concrètes plutôt qu'en jugements ("Pourquoi pas plutôt X ?" plutôt que "C'est mal fait").

### 05 - Branch protection rules

**Explication** — Règles appliquées à une branche (typiquement `main`) : exiger au moins N reviews approuvées avant fusion, exiger que les checks CI passent, interdire le push direct ou le force-push.

**Bonne pratique** : protéger systématiquement la branche principale avec au minimum "require status checks to pass" — évite qu'un push direct (volontaire ou accidentel) casse le déploiement.

### 06 - GitHub Actions : premier workflow

**Explication** — Un workflow est un fichier YAML dans `.github/workflows/`, déclenché par un événement (`on:`), composé de jobs eux-mêmes composés de steps.

```yaml
# .github/workflows/ci.yml
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

**Bonne pratique** : déclencher le workflow à la fois sur `push` (branche principale) et sur `pull_request` — cela permet de voir l'état des checks directement sur la PR avant fusion.

### 07 - README et documentation d'un dépôt

**Explication** — Le `README.md` à la racine s'affiche automatiquement sur la page du dépôt ; c'est le premier point de contact d'un contributeur ou utilisateur (installation, usage, contribution).

## 4. Concepts intermédiaires 🟡

- **Matrix builds** : exécuter le même job avec plusieurs combinaisons de paramètres (versions de langage, OS) en parallèle.

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node-version }}
```

- **Secrets** : valeurs sensibles (tokens, clés API) stockées chiffrées au niveau du dépôt ou d'un environment, exposées aux workflows via `${{ secrets.MON_SECRET }}` — jamais en clair dans le YAML.
- **CODEOWNERS** : fichier `.github/CODEOWNERS` qui assigne automatiquement des relecteurs obligatoires selon les fichiers modifiés par une PR.
- **GitHub Projects** : tableau kanban lié aux Issues/PR du dépôt (ou de plusieurs dépôts), avec vues personnalisables (tableau, liste, roadmap).
- **Draft Pull Requests** : PR marquée "brouillon", visible mais non mergeable, pour partager un travail en cours sans déclencher de demande de review formelle.
- **Stratégies de fusion** : *merge commit* (conserve tout l'historique de la branche), *squash and merge* (compresse la branche en un seul commit sur `main`), *rebase and merge* (rejoue les commits un par un sans commit de merge) — le choix impacte la lisibilité de l'historique.
- **Templates d'Issues et de PR** : fichiers dans `.github/ISSUE_TEMPLATE/` et `.github/pull_request_template.md` qui pré-remplissent la structure attendue, pour standardiser les rapports de bug et les descriptions de PR.

## 5. Concepts avancés 🟠🔴

- **Reusable workflows** : un workflow peut en appeler un autre via `workflow_call`, pour factoriser une pipeline commune entre plusieurs dépôts sans dupliquer le YAML.

```yaml
jobs:
  call-shared-ci:
    uses: mon-org/shared-workflows/.github/workflows/ci.yml@main
    with:
      node-version: '20'
```

- **Composite actions** : regrouper une séquence de steps réutilisable dans une action custom (`action.yml`), utilisable comme un `uses:` classique dans n'importe quel workflow.
- **Environments et required reviewers** : un `environment:` (ex. `production`) peut exiger une approbation manuelle avant que le job de déploiement ne s'exécute — un gate humain intégré à la pipeline.
- **Semantic release et tagging automatique** : analyser les messages de commit (Conventional Commits) pour déterminer automatiquement la version sémantique suivante, générer le changelog et le tag Git à la fusion sur `main` (ex. via `semantic-release`).
- **Self-hosted runners** : exécuter les jobs sur une machine que l'on contrôle (accès réseau interne, matériel spécifique, coût) plutôt que sur les runners GitHub hébergés.
- **Sécurité de la plateforme** : Dependabot (alertes et PR automatiques de mise à jour de dépendances vulnérables), CodeQL (analyse statique de sécurité intégrée), commits signés obligatoires sur une branche protégée — voir [`../security/`](../security/).
- **Déclenchement conditionnel par chemin (path filters)** : dans un monorepo, limiter un workflow à se déclencher uniquement quand certains chemins changent (`paths: ['packages/api/**']`) — évite de relancer toute la CI pour un changement isolé.

## 6. Commandes / syntaxe à connaître

```bash
gh repo clone owner/repo             # cloner via la CLI GitHub
gh pr create --base main --head branche --title "..." --body "..."
gh pr checkout 42                     # récupérer localement la PR #42
gh pr merge 42 --squash               # fusionner une PR
gh issue create --title "..." --body "..."
gh run list                           # lister les exécutions de workflows récentes
gh run watch                          # suivre l'exécution d'un run en direct
```

```yaml
on: { push: { branches: [main] }, pull_request: {} }
jobs:
  job-id:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "commande"
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Mise en place d'un flux de collaboration complet sur un petit dépôt**

- Un workflow `.github/workflows/ci.yml` qui installe les dépendances et lance les tests à chaque push et pull request.
- Une règle de protection sur `main` exigeant que ce check passe et qu'au moins une review soit approuvée avant fusion.
- Un template de Pull Request (`.github/pull_request_template.md`) et un template d'Issue de bug (`.github/ISSUE_TEMPLATE/bug_report.md`).
- Un fichier `CODEOWNERS` assignant automatiquement un relecteur sur les fichiers du dossier `src/`.
- Un second workflow, déclenché uniquement à la fusion sur `main`, qui crée automatiquement un tag de version.
- Bonus : transformer le workflow de CI en *reusable workflow* et l'appeler depuis un second dépôt fictif.

## Checklist

- [ ] Comprendre les fondamentaux (repository, fork/clone, Pull Requests, Issues)
- [ ] Savoir ouvrir et faire évoluer une Pull Request jusqu'à la fusion
- [ ] Maîtriser la syntaxe principale d'un workflow GitHub Actions
- [ ] Comprendre les concepts importants (secrets, matrix builds, stratégies de fusion)
- [ ] Savoir utiliser la CLI `gh` au quotidien
- [ ] Connaître les bonnes pratiques (PR petites et focalisées, protection de branche, review constructive)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (reusable workflows, environments protégés, sécurité de la plateforme)

## 10. Ressources

- [Documentation officielle GitHub](https://docs.github.com/) — référence complète (dépôts, PR, Actions, sécurité).
- [GitHub Actions — documentation](https://docs.github.com/actions) — syntaxe des workflows, actions du marketplace.
- [GitHub CLI (`gh`)](https://cli.github.com/) — documentation de l'outil en ligne de commande.
- Il n'existe pas de roadmap.sh dédiée à GitHub spécifiquement ; voir [roadmap.sh — DevOps](https://roadmap.sh/devops) pour le contexte plus large (CI/CD, collaboration, outillage).
