# Git — Solutions Niveau 2

```bash
# 1. Squash de commits sales
git switch -c feature/x
echo "a" > f.txt && git add f.txt && git commit -m "wip 1"
echo "b" >> f.txt && git add f.txt && git commit -m "wip 2"
echo "c" >> f.txt && git add f.txt && git commit -m "fix typo"

git rebase -i HEAD~3
# Dans l'éditeur : garder "pick" sur le premier commit,
# remplacer "pick" par "squash" (ou "s") sur les deux suivants.
# Un second écran permet de réécrire le message final, ex :
# "Add f.txt with initial content"
git log --oneline   # 1 seul commit désormais
```

```bash
# 2. Conflit volontaire
git switch main
echo "ligne originale" > shared.txt && git add shared.txt && git commit -m "Add shared.txt"

git switch -c branch-a
echo "modif A" > shared.txt && git add shared.txt && git commit -m "Modify from A"

git switch main
git switch -c branch-b
echo "modif B" > shared.txt && git add shared.txt && git commit -m "Modify from B"

git switch main
git merge branch-a          # OK, fast-forward ou merge simple
git merge branch-b          # CONFLICT

# Ouvrir shared.txt, il contient :
# <<<<<<< HEAD
# modif A
# =======
# modif B
# >>>>>>> branch-b
# → éditer pour garder le sens des deux, ex :
# modif A et modif B combinées
git add shared.txt
git commit   # finalise le merge
```

```bash
# 3. Stash
echo "travail en cours" >> f.txt
git stash push -m "wip formulaire"
git switch autre-branche
git commit -am "commit sur autre-branche" --allow-empty
git switch -            # retour sur la branche précédente
git stash pop
```

```bash
# 4. Cherry-pick
git log --oneline main             # repérer un hash au milieu, ex: a1b2c3d
git switch -c hotfix <commit-ancien>
git cherry-pick a1b2c3d
```

```bash
# 5. Reset --soft vs Revert
git switch -c demo-reset
echo "x" >> f.txt && git add f.txt && git commit -m "Commit à annuler"
git reset --soft HEAD~1
git log --oneline   # le commit a disparu de l'historique, changements restent en staging

git switch main
git switch -c demo-revert
echo "y" >> f.txt && git add f.txt && git commit -m "Commit à annuler"
git revert HEAD --no-edit
git log --oneline   # le commit original reste visible + un nouveau commit "Revert ..." apparaît
```
**Différence observée** : `reset --soft` réécrit l'historique (le commit annulé disparaît de `git log`, comme s'il n'avait jamais existé) — dangereux si déjà partagé. `revert` préserve l'historique complet et ajoute un commit inverse — sûr sur du code déjà publié/déployé.

```bash
# 6. Alias
git config --global alias.lg "log --oneline --graph --all --decorate"
git lg
```

```bash
# 7. Récupération via reflog
echo "travail important" >> f.txt && git add f.txt && git commit -m "Travail important"
git reset --hard HEAD~1     # "perd" le commit
git reflog                  # repérer la ligne juste avant le reset, ex: HEAD@{1}
git reset --hard HEAD@{1}   # ou : git branch recuperation HEAD@{1}
git log --oneline           # le commit "Travail important" est de retour
```
