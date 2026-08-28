# Git

## 1. Introduction

Git est un **système de contrôle de version distribué (DVCS)** : il enregistre l'historique complet d'un projet sous forme d'instantanés (snapshots), permet de travailler en parallèle sur plusieurs lignes de développement (branches) et de fusionner ce travail sans dépendre d'un serveur central.

**Problèmes qu'il résout :**
- Perdre du code / ne pas pouvoir revenir en arrière.
- Travailler à plusieurs sur les mêmes fichiers sans s'écraser mutuellement.
- Savoir *qui* a changé *quoi*, *quand* et *pourquoi*.
- Isoler le développement d'une fonctionnalité ou d'un correctif du code stable (branches).

**Où il se situe dans l'architecture :** Git est un outil **local**, indépendant de toute plateforme. `github/` (dossier séparé) documente GitHub, qui est une plateforme d'hébergement + collaboration (PR, Issues, CI) **construite autour** de Git, mais pas Git lui-même. GitLab, Bitbucket, ou un simple serveur `git` nu fonctionnent avec les mêmes commandes Git décrites ici.

**Avantages :**
- Distribué : chaque clone contient tout l'historique, donc pas de point de panne unique, travail possible hors-ligne.
- Branches légères et rapides (contrairement à SVN/CVS).
- Intégrité garantie par hachage SHA-1/SHA-256 de tout le contenu.

**Limites :**
- Mauvais historiquement pour les gros fichiers binaires (nécessite Git LFS).
- Courbe d'apprentissage réelle sur les commandes avancées (rebase interactif, reflog...).
- Un historique mal géré (mauvais messages, commits monstres) reste lisible par la machine mais illisible par les humains — Git ne force aucune discipline.

---

## 2. Prérequis

- Ligne de commande (naviguer dans les dossiers, exécuter une commande, lire une sortie terminal).
- Notion de fichier texte vs binaire.
- Un éditeur de texte pour résoudre les conflits.

---

## 3. Rappel des bases 🟢

### Initialiser / cloner un dépôt

- **Explication** : `init` crée un nouveau dépôt vide dans le dossier courant (crée un dossier caché `.git/`) ; `clone` récupère un dépôt existant (historique complet) depuis une URL.
- **Syntaxe**
  ```bash
  git init
  git clone https://github.com/user/repo.git
  git clone https://github.com/user/repo.git mon-dossier   # renommer le dossier local
  ```
- **Cas d'usage** : `init` pour démarrer un nouveau projet, `clone` pour rejoindre un projet existant.
- **Erreur fréquente** : lancer `git init` dans un dossier qui contient déjà un `.git/` parent (ex: son `$HOME`) → tout le disque devient "suivi". Vérifier `git status` juste après.
- **Bonne pratique** : toujours vérifier avec `git remote -v` après un clone que l'URL distante est la bonne (HTTPS vs SSH).

### La zone de staging (index)

- **Explication** : Git a trois zones : *working directory* (fichiers sur disque), *staging area / index* (ce qui sera inclus dans le prochain commit), *repository* (historique des commits). C'est la subtilité n°1 par rapport à d'autres VCS : on choisit précisément quoi committer, pas "tout ce qui a changé".
- **Syntaxe**
  ```bash
  git add fichier.js         # ajoute un fichier au staging
  git add .                  # ajoute tout le dossier courant (récursif)
  git add -p                 # ajoute par "hunks" (morceaux de diff), interactif
  git restore --staged fichier.js   # retire du staging sans perdre les modifs
  ```
- **Cas d'usage** : découper un gros changement en plusieurs commits logiques via `git add -p`.
- **Erreur fréquente** : `git add .` par réflexe, qui embarque des fichiers non voulus (`.env`, `node_modules/`, fichiers de debug).
- **Bonne pratique** : toujours faire `git status` puis `git diff --staged` avant de committer, pour relire exactement ce qui va partir.

### Commit

- **Explication** : un commit est un instantané immuable du projet à un instant T, identifié par un hash SHA. Il référence son(ses) commit(s) parent(s), formant un graphe (le plus souvent une chaîne, un arbre en cas de branches).
- **Syntaxe**
  ```bash
  git commit -m "message"
  git commit -am "message"      # add + commit, mais SEULEMENT les fichiers déjà suivis (tracked)
  git commit --amend            # modifie le dernier commit (message et/ou contenu)
  ```
- **Cas d'usage** : `--amend` pour corriger une typo dans le dernier message ou ajouter un fichier oublié, **avant** d'avoir push.
- **Erreur fréquente** : utiliser `--amend` sur un commit déjà poussé et partagé → réécrit l'historique distant, casse le travail des collègues qui ont basé du travail dessus.
- **Bonne pratique** : messages de commit au format impératif ("Add", "Fix", pas "Added"/"Fixed"), un commit = un changement logique cohérent (voir [Conventional Commits](https://www.conventionalcommits.org/)).

### Status, diff, log

- **Explication** : outils de lecture de l'état du dépôt, sans jamais rien modifier.
- **Syntaxe**
  ```bash
  git status                 # état des fichiers (staged / modifiés / non suivis)
  git diff                   # diff working dir vs staging
  git diff --staged          # diff staging vs dernier commit
  git log                    # historique des commits
  git log --oneline --graph --all   # vue condensée avec branches
  git log -p -- fichier.js   # historique détaillé d'un fichier précis
  git blame fichier.js       # qui a modifié chaque ligne, et dans quel commit
  ```
- **Cas d'usage** : `git blame` pour retrouver le contexte d'une ligne suspecte avant de la modifier.
- **Erreur fréquente** : oublier `--staged` et croire que `git diff` montre tout (il ignore ce qui est déjà en staging).
- **Bonne pratique** : `git log --oneline --graph --all --decorate` en alias pour visualiser rapidement l'état complet du dépôt.

### Branches

- **Explication** : une branche est simplement un pointeur mobile vers un commit. Créer une branche est quasi instantané et ne duplique aucun fichier (contrairement à SVN).
- **Syntaxe**
  ```bash
  git branch                     # lister les branches locales
  git branch nom-branche         # créer une branche (sans y basculer)
  git switch nom-branche         # basculer dessus (moderne, remplace "checkout")
  git switch -c nom-branche      # créer + basculer en une commande
  git branch -d nom-branche      # supprimer (refuse si non mergée)
  git branch -D nom-branche      # supprimer en forçant
  ```
- **Cas d'usage** : une branche par fonctionnalité/correctif (`feature/panier`, `fix/login-bug`), jamais développer directement sur `main`.
- **Erreur fréquente** : confondre `git checkout` (commande fourre-tout historique : change de branche, restaure des fichiers, détache HEAD...) avec les commandes modernes plus explicites `git switch` (branches) et `git restore` (fichiers).
- **Bonne pratique** : nommer les branches avec un préfixe explicite (`feature/`, `fix/`, `chore/`) et les supprimer une fois mergées.

### Merge (fusion simple)

- **Explication** : intègre les commits d'une branche dans une autre. En "fast-forward" si l'historique cible n'a pas divergé (le pointeur avance simplement) ; en "merge commit" sinon (crée un commit avec deux parents).
- **Syntaxe**
  ```bash
  git switch main
  git merge feature/panier
  git merge --no-ff feature/panier   # force un commit de merge même si fast-forward possible
  ```
- **Cas d'usage** : intégrer une feature terminée dans `main`.
- **Erreur fréquente** : merger `main` dans sa branche de feature à répétition "pour rester à jour" sans réfléchir → historique illisible avec des dizaines de merge commits.
- **Bonne pratique** : `--no-ff` sur les branches partagées pour garder une trace explicite de "quand la feature X a été intégrée", même si techniquement un fast-forward était possible.

### Remotes (dépôts distants)

- **Explication** : un remote est un alias vers l'URL d'un autre dépôt Git (souvent nommé `origin`). `fetch` télécharge les nouveaux commits sans les appliquer, `pull` = `fetch` + `merge` (ou `rebase` selon config).
- **Syntaxe**
  ```bash
  git remote -v                       # lister les remotes
  git remote add origin <url>
  git fetch origin                    # récupère sans fusionner
  git pull                            # récupère + fusionne (fetch + merge)
  git pull --rebase                   # récupère + rebase (historique plus propre)
  git push origin nom-branche
  git push -u origin nom-branche      # push + mémorise le lien de tracking
  ```
- **Cas d'usage** : `git fetch` seul pour inspecter les changements distants (`git log origin/main`) avant de décider comment les intégrer.
- **Erreur fréquente** : faire `git pull` sans savoir si ça va merger ou rebaser (dépend de `pull.rebase` en config), et être surpris par un merge commit inattendu.
- **Bonne pratique** : configurer `git config --global pull.rebase true` pour un historique linéaire par défaut, et toujours `fetch` avant de push sur une branche partagée.

### `.gitignore`

- **Explication** : fichier listant les patterns de fichiers/dossiers à ne jamais suivre (dépendances, secrets, artefacts de build).
- **Syntaxe**
  ```gitignore
  node_modules/
  .env
  *.log
  dist/
  ```
- **Cas d'usage** : exclure `vendor/`, `node_modules/`, `.env`, les fichiers d'IDE (`.idea/`, `.vscode/` selon convention d'équipe).
- **Erreur fréquente** : ajouter un fichier à `.gitignore` alors qu'il est **déjà suivi** → ça ne l'ignore pas rétroactivement (`git rm --cached fichier` est nécessaire en plus).
- **Bonne pratique** : utiliser [gitignore.io](https://www.toptal.com/developers/gitignore) pour générer un `.gitignore` de base par stack, et ne jamais committer de secrets même dans un commit "corrigé" ensuite (voir Concepts avancés > réécriture d'historique).

### Tags

- **Explication** : un tag est un pointeur *fixe* (contrairement à une branche) vers un commit précis, typiquement utilisé pour marquer une version publiée.
- **Syntaxe**
  ```bash
  git tag v1.0.0                       # tag léger
  git tag -a v1.0.0 -m "Version 1.0.0" # tag annoté (recommandé, avec métadonnées)
  git push origin v1.0.0               # les tags ne sont PAS poussés automatiquement
  git push origin --tags
  ```
- **Cas d'usage** : versionning sémantique (`v1.2.0`) pour déclencher un déploiement ou une release.
- **Erreur fréquente** : oublier que `git push` seul ne pousse pas les tags.
- **Bonne pratique** : préférer les tags annotés (`-a`) aux tags légers, ils stockent auteur/date/message.

---

## 4. Concepts intermédiaires 🟡

### Merge vs Rebase

La question la plus mal comprise de Git.

| | Merge | Rebase |
|---|---|---|
| Effet | Crée un commit de fusion, préserve l'historique tel qu'il s'est réellement passé | Réécrit les commits de la branche comme s'ils avaient été créés après les derniers commits de la cible |
| Historique | Non-linéaire, avec des branches visibles | Linéaire, "propre" |
| Risque | Aucun (opération additive) | Réécrit les hash des commits → **jamais sur une branche déjà partagée/push** sans coordination |
| Commande | `git merge feature` | `git rebase main` (depuis `feature`) |

**Règle d'or** : *rebase local, merge public.* On peut rebaser librement sa propre branche de feature avant de l'ouvrir en PR ; on ne rebase jamais une branche que d'autres ont déjà basée dessus (ça force tout le monde à un `push --force` en cascade).

### Résolution de conflits

- **Explication** : un conflit survient quand Git ne peut pas fusionner automatiquement deux versions d'une même zone de fichier. Git marque les zones avec `<<<<<<<`, `=======`, `>>>>>>>`.
- **Syntaxe**
  ```bash
  git merge feature/x
  # CONFLICT (content): Merge conflict in fichier.js
  # → éditer le fichier, choisir/fusionner le bon contenu, retirer les marqueurs
  git add fichier.js
  git commit             # (ou "git rebase --continue" si on était en rebase)
  git merge --abort       # annuler et revenir à l'état avant le merge
  git rebase --abort      # idem pour un rebase
  ```
- **Cas d'usage** : deux personnes modifient la même ligne d'un fichier de config.
- **Erreur fréquente** : résoudre "à l'aveugle" en gardant sa version sans lire ce que l'autre a changé, ou committer avec les marqueurs `<<<<<<<` encore présents.
- **Bonne pratique** : utiliser un outil visuel (`git mergetool`, ou l'éditeur/IDE) et **toujours retester** après résolution — un merge "réussi" côté Git peut casser le code fonctionnellement.

### Stash

- **Explication** : range temporairement les modifications en cours (working dir + staging) sans les committer, pour revenir à un état propre.
- **Syntaxe**
  ```bash
  git stash                 # range les modifs
  git stash push -m "wip: formulaire"
  git stash list
  git stash pop              # ré-applique le dernier stash et le supprime de la pile
  git stash apply stash@{1}  # ré-applique sans supprimer
  git stash drop stash@{0}
  ```
- **Cas d'usage** : on doit changer de branche en urgence (hotfix) alors qu'on a du travail non fini et non committable.
- **Erreur fréquente** : accumuler des stashs oubliés qui deviennent obsolètes et provoquent des conflits illisibles des mois plus tard.
- **Bonne pratique** : toujours nommer ses stash (`-m`), et préférer un commit "WIP" sur une branche temporaire si le travail doit survivre longtemps.

### Cherry-pick

- **Explication** : applique un commit précis (par son hash) sur la branche courante, sans fusionner toute la branche source.
- **Syntaxe**
  ```bash
  git cherry-pick <hash>
  git cherry-pick <hash1> <hash2>
  git cherry-pick --continue   # après résolution d'un conflit
  ```
- **Cas d'usage** : porter un correctif de sécurité d'une branche `main` vers une branche de release `v1.x` sans tout fusionner.
- **Erreur fréquente** : cherry-picker un commit qui dépend d'autres commits non inclus → code cassé silencieusement.
- **Bonne pratique** : rester exceptionnel, documenter dans le message pourquoi le commit a été cherry-pické (Git ajoute déjà `(cherry picked from commit ...)` avec `-x`).

### Reset vs Revert

- **Explication** : `reset` déplace le pointeur de branche (et éventuellement le contenu) vers un commit antérieur — **réécrit l'historique local**. `revert` crée un **nouveau** commit qui annule les changements d'un commit précédent — **préserve l'historique**.
- **Syntaxe**
  ```bash
  git reset --soft <hash>    # déplace HEAD, garde staging + working dir intacts
  git reset --mixed <hash>   # déplace HEAD + vide le staging (défaut), garde working dir
  git reset --hard <hash>    # déplace HEAD + staging + working dir : PERTE DE DONNÉES non commitées
  git revert <hash>          # nouveau commit qui inverse <hash>
  ```
- **Cas d'usage** : `reset --soft` pour regrouper plusieurs commits locaux avant de re-committer proprement ; `revert` pour annuler un commit déjà en production (garde la traçabilité).
- **Erreur fréquente** : `git reset --hard` sur une branche partagée déjà poussée → perte de commits pour tout le monde au prochain pull (sauf reflog, voir avancé).
- **Bonne pratique** : **jamais `reset --hard`/`push --force` sur une branche partagée** ; utiliser `revert` dans ce cas. `reset` reste sûr sur du travail 100% local non poussé.

### Aliases et configuration

- **Syntaxe**
  ```bash
  git config --global user.name "Nom"
  git config --global user.email "email@example.com"
  git config --global alias.st status
  git config --global alias.lg "log --oneline --graph --all --decorate"
  git config --global core.editor "code --wait"
  ```
- **Bonne pratique** : configurer un `.gitconfig` global versionné personnellement (dotfiles), pour retrouver ses alias sur toute nouvelle machine.

---

## 5. Concepts avancés 🟠🔴

### Internals : objects, refs, HEAD

Git n'est fondamentalement qu'une base de données clé-valeur de type content-addressable, plus une couche de pointeurs.

- **4 types d'objets** (dans `.git/objects/`, identifiés par le hash SHA de leur contenu) :
  - `blob` : contenu brut d'un fichier (pas de nom, juste les octets).
  - `tree` : un annuaire — associe des noms de fichiers/dossiers à des blobs/trees.
  - `commit` : pointe vers un `tree` (l'état du projet), un ou plusieurs commits parents, auteur, message.
  - `tag` : (annoté) pointe vers un commit avec métadonnées supplémentaires.
- **Refs** : simples fichiers texte contenant un hash. `.git/refs/heads/main` = une branche, `.git/HEAD` = pointeur vers la branche courante (ou un commit précis en mode "detached HEAD").
- **Commande d'exploration** :
  ```bash
  git cat-file -p <hash>      # affiche le contenu d'un objet quel qu'il soit
  git rev-parse HEAD          # hash du commit courant
  git ls-tree HEAD            # contenu du tree racine du commit courant
  ```
- **Pourquoi c'est important** : comprendre que **rien n'est jamais "modifié" en place** — un commit crée toujours de nouveaux objets — explique pourquoi `reset`/`rebase` sont "sûrs" (les anciens objets restent en base, récupérables via reflog) tant qu'ils ne sont pas garbage-collectés.

### Reflog : le filet de sécurité

- **Explication** : Git journalise localement **tous** les déplacements de `HEAD` (commits, resets, rebases, checkouts...), même ceux qui semblent avoir "supprimé" des commits de l'historique visible.
- **Syntaxe**
  ```bash
  git reflog
  git reset --hard HEAD@{2}     # revenir à un état antérieur retrouvé dans le reflog
  git branch recover-branch <hash-perdu>
  ```
- **Cas d'usage** : récupérer des commits après un `reset --hard` malheureux ou un rebase raté.
- **Limite** : le reflog est **local** (pas poussé, pas partagé) et expire par défaut après 90 jours (commits atteignables) / 30 jours (unreachable).
- **Bonne pratique** : en cas d'erreur, ne pas paniquer — sauf `git gc --prune=now` juste après, presque tout est récupérable via `reflog`.

### Rebase interactif

- **Explication** : permet de réécrire l'historique d'une série de commits locaux : réordonner, fusionner (squash/fixup), reformuler des messages, supprimer des commits.
- **Syntaxe**
  ```bash
  git rebase -i HEAD~5          # les 5 derniers commits
  # ouvre un éditeur avec : pick / reword / edit / squash / fixup / drop
  ```
- **Cas d'usage** : nettoyer une branche de feature (10 commits "wip", "fix typo"...) en 2-3 commits logiques avant d'ouvrir une PR.
- **Erreur fréquente** : rebaser interactivement une branche déjà partagée sans prévenir → collègues bloqués avec des conflits en cascade.
- **Bonne pratique** : squash/fixup avant de pousser pour la première fois ; une fois en revue par d'autres, préférer ajouter des commits plutôt que réécrire (sauf accord explicite de l'équipe, ex: "toujours squash-merge en fin de PR").

### Réécriture d'historique lourde

- **Explication** : pour retirer un fichier/secret de **tout** l'historique (pas juste HEAD), `git rebase -i` ne suffit pas — il faut réécrire tous les commits concernés.
- **Outils** : `git filter-repo` (officiellement recommandé, remplace l'ancien `filter-branch`, trop lent et dangereux).
  ```bash
  git filter-repo --path secrets.env --invert-paths   # supprime le fichier de tout l'historique
  ```
- **Cas d'usage** : un secret (clé API, mot de passe) a été commité par erreur.
- **Point critique** : réécrire l'historique change **tous** les hash en aval → force-push obligatoire, et **le secret doit être considéré comme compromis et révoqué**, la réécriture d'historique ne suffit pas si le dépôt était déjà accessible (clones existants, caches).

### Hooks

- **Explication** : scripts exécutés automatiquement à certains événements Git (`pre-commit`, `commit-msg`, `pre-push`...), stockés dans `.git/hooks/` (non versionnés par défaut).
- **Cas d'usage** : lancer un linter avant chaque commit, valider le format du message de commit, lancer les tests avant un push.
- **Bonne pratique pro** : ne pas dépendre de `.git/hooks/` (non partagé par clone) — utiliser un outil comme [Husky](https://typicode.github.io/husky/) (JS) qui versionne la config des hooks dans le repo lui-même.

### Bisect

- **Explication** : recherche binaire automatisée dans l'historique pour trouver le commit qui a introduit un bug.
- **Syntaxe**
  ```bash
  git bisect start
  git bisect bad                # le commit courant est cassé
  git bisect good v1.0.0        # ce commit était bon
  # Git checkout un commit intermédiaire → tester → répéter :
  git bisect good  # ou
  git bisect bad
  git bisect reset              # terminer
  ```
- **Cas d'usage** : régression détectée en prod, des dizaines de commits candidats — `bisect` réduit ça à `log2(n)` tests. Peut être automatisé avec `git bisect run ./test.sh`.

### Worktrees

- **Explication** : permet d'avoir plusieurs branches "checkout" simultanément dans des dossiers séparés, sans dupliquer le `.git/`.
- **Syntaxe**
  ```bash
  git worktree add ../hotfix-dir fix/urgent-bug
  git worktree list
  git worktree remove ../hotfix-dir
  ```
- **Cas d'usage** : traiter un hotfix urgent sans devoir stash/committer le travail en cours dans le dossier principal.

### Performance sur gros dépôts

- **Shallow clone** : `git clone --depth 1` (historique tronqué, utile en CI où l'historique complet n'est pas nécessaire).
- **Sparse checkout** : `git sparse-checkout set dossier1 dossier2` (ne récupère qu'une partie de l'arborescence, utile en monorepo).
- **Partial clone** : `git clone --filter=blob:none` (récupère les métadonnées sans le contenu des fichiers, téléchargé à la demande).

### Signature des commits

- **Explication** : signer cryptographiquement les commits (GPG ou clé SSH) pour prouver leur authenticité (affiché "Verified" sur GitHub).
- **Syntaxe**
  ```bash
  git config --global commit.gpgsign true
  git config --global user.signingkey <key-id>
  git commit -S -m "message"
  ```
- **Cas d'usage** : projets open-source sensibles, conformité (supply chain security).

---

## 6. Commandes / syntaxe à connaître

```bash
# Configuration
git config --global user.name "Nom"
git config --global user.email "email@example.com"

# Cycle de base
git status
git add <fichier|.>
git commit -m "message"
git push
git pull --rebase

# Branches
git switch -c ma-branche
git branch -d ma-branche
git merge ma-branche
git rebase main

# Historique
git log --oneline --graph --all
git diff
git diff --staged
git blame <fichier>

# Annuler / corriger
git commit --amend
git reset --soft HEAD~1
git revert <hash>
git reflog

# Temporaire
git stash
git stash pop

# Debug
git bisect start
git cherry-pick <hash>
```

---

## 7. Exercices

Les énoncés sont dans [`exercices/`](exercices/), les corrections dans [`solutions/`](solutions/) (à consulter seulement après avoir essayé).

- **Niveau 1 — Bases** : [`exercices/niveau-1.md`](exercices/niveau-1.md)
- **Niveau 2 — Intermédiaire** : [`exercices/niveau-2.md`](exercices/niveau-2.md)
- **Niveau 3 — Avancé** : [`exercices/niveau-3.md`](exercices/niveau-3.md)

---

## 8. Mini-projet

**Simuler un workflow d'équipe complet sur un dépôt local.**

1. Crée un dépôt `git init workflow-demo`, fais un premier commit avec un fichier `app.js`.
2. Crée deux branches `feature/login` et `feature/panier` à partir de `main`, fais 2-3 commits sur chacune en modifiant les **mêmes lignes** d'un fichier commun (pour provoquer un conflit plus tard).
3. Sur `feature/login`, fais 4-5 petits commits "sales" (`wip`, `fix typo`, `oops`) puis nettoie-les avec un `rebase -i` (squash) en 1 seul commit propre avant de merger dans `main`.
4. Merge `feature/panier` dans `main` : résous le conflit qui apparaît proprement, en gardant le sens des deux modifications.
5. Simule une erreur : fais un `reset --hard` qui perd un commit important, puis récupère-le via `git reflog`.
6. Ajoute un tag annoté `v1.0.0` sur le commit final.
7. Bonus : ajoute un hook `pre-commit` (dans `.git/hooks/`) qui refuse le commit si le mot `console.log` est présent dans les fichiers stagés.

---

## Checklist

- [ ] Comprendre les fondamentaux (init, add, commit, branch, merge, remote)
- [ ] Savoir créer un projet et le connecter à un remote
- [ ] Maîtriser la syntaxe principale (status, diff, log, branch, merge, push/pull)
- [ ] Comprendre les concepts importants (merge vs rebase, stash, reset vs revert, conflits)
- [ ] Savoir debugger (reflog, bisect, blame)
- [ ] Connaître les bonnes pratiques (messages de commit, .gitignore, pas de force-push sur du partagé)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (internals, rebase interactif, réécriture d'historique, hooks, worktrees)

---

## 10. Ressources

- [Documentation officielle Git](https://git-scm.com/doc) — référence complète de chaque commande.
- [Pro Git Book](https://git-scm.com/book/fr/v2) — disponible en français, gratuit, la meilleure ressource pour approfondir les internals.
- [git-scm.com/docs/gitglossary](https://git-scm.com/docs/gitglossary) — glossaire officiel des termes (utile pour lever les ambiguïtés).
- [roadmap.sh/git-github](https://roadmap.sh/git-github) — seule la partie **Git pur** (hors PR/CI/collaboration GitHub) est couverte dans ce dossier ; le reste est traité dans `github/`.
