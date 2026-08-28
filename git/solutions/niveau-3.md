# Git — Solutions Niveau 3

### 1. Exploration des internals

```bash
git rev-parse HEAD                       # hash du commit courant
git cat-file -p HEAD                     # affiche : tree <hash>, parent <hash>, author, message
git cat-file -p <hash-du-tree>           # liste les entrées (mode, type, hash, nom) du tree racine
git cat-file -p <hash-du-blob-fichier>   # affiche le contenu brut du fichier
diff <(git cat-file -p <hash-du-blob-fichier>) fichier.txt   # doit être vide (identique)
```
Un commit ne contient jamais directement de fichiers : il pointe vers un `tree` (l'état de l'arborescence), qui pointe vers des `blob` (contenu) et d'autres `tree` (sous-dossiers). Le contenu d'un blob est strictement le contenu du fichier, sans nom ni métadonnées — le nom vit dans le `tree` qui le référence.

### 2. Secret accidentellement commité

```bash
echo "API_KEY=fake12345" > secret.env
git add secret.env && git commit -m "Add secret.env"
echo "later change" >> f.txt && git add f.txt && git commit -m "Unrelated commit"

# Installer git-filter-repo si nécessaire (pip install git-filter-repo)
git filter-repo --path secret.env --invert-paths

git log --all --oneline -- secret.env   # aucun résultat : le fichier n'existe plus dans AUCUN commit
```
Important en conditions réelles : `filter-repo` réécrit tous les hash en aval → nécessite un `push --force` coordonné avec l'équipe, et **la clé doit être révoquée côté fournisseur** (elle a pu être clonée/indexée avant le nettoyage).

### 3. Bisect automatisé

```bash
# test.sh (exit 0 = bon, exit 1 = mauvais)
#!/bin/bash
result=$(node -e "console.log(require('./calc').add(2,2))")
[ "$result" = "4" ] && exit 0 || exit 1
```
```bash
git bisect start
git bisect bad HEAD
git bisect good <hash-du-tout-premier-commit>
git bisect run ./test.sh
# Git affiche : "<hash> is the first bad commit"
git bisect reset
```

### 4. Rebase interactif avancé

```bash
git rebase -i HEAD~6
```
Dans l'éditeur, en partant d'une liste `pick c1..c6` (du plus ancien en haut) :
- Réordonner : couper/coller la ligne du commit à déplacer.
- Fusionner : remplacer `pick` par `fixup` sur le commit à absorber, placé juste après le commit qu'il complète (ne garde que le message du commit `pick`).
- Reformuler : remplacer `pick` par `reword` sur le commit concerné → un second écran s'ouvre pour éditer son message.

Une seule session de rebase applique les trois opérations dans l'ordre où Git rencontre les commits.

### 5. Worktrees

```bash
git worktree add ../hotfix-dir -b hotfix/urgent main
cd ../hotfix-dir
# travail isolé sur le hotfix, le dossier principal garde ses modifications non commitées intactes
git worktree list
git worktree remove ../hotfix-dir   # une fois terminé
```

### 6. Hook de qualité

```bash
# .git/hooks/pre-commit (rendre exécutable : chmod +x)
#!/bin/bash
files=$(git diff --cached --name-only --diff-filter=ACM)
blocked=0
for f in $files; do
  matches=$(grep -n -E "TODO|console\.log" "$f")
  if [ -n "$matches" ]; then
    echo "Commit bloqué : '$f' contient TODO/console.log :"
    echo "$matches"
    blocked=1
  fi
done
exit $blocked
```
En conditions professionnelles : ce script vivrait versionné dans le repo (ex: `scripts/pre-commit.sh`) et serait installé via [Husky](https://typicode.github.io/husky/), car `.git/hooks/` n'est jamais partagé par clone.

### 7. Scénario de production

Stratégie correcte : **`git revert`**, pas de réécriture d'historique.

```bash
git bisect ...            # identifie <hash-fautif>
git revert <hash-fautif> --no-edit
git push origin main
```
Justification : le commit fautif est déjà mergé, déployé, potentiellement basé par d'autres branches, et peut apparaître dans des logs/audits externes. Réécrire l'historique (`reset --hard` + `push --force`, ou `filter-repo`) casserait tous les clones existants et effacerait la traçabilité de ce qui s'est réellement passé — inacceptable sur une branche partagée en production. `revert` ajoute un commit qui neutralise les changements tout en conservant une trace explicite et auditable de l'incident et de sa correction.
