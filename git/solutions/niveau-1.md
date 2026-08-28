# Git — Solutions Niveau 1

```bash
# 1.
mkdir exo-git-1 && cd exo-git-1
git init
git status   # confirme "On branch main / No commits yet"

# 2.
echo "Première note" > notes.txt
git add notes.txt
git commit -m "Add notes.txt with initial content"

# 3.
echo "Deuxième ligne" >> notes.txt
git diff                    # montre la modif non stagée

# 4.
git add notes.txt
git diff --staged           # montre la modif stagée
git commit -m "Add second line to notes.txt"

# 5.
git log --oneline

# 6.
git switch -c feature/notes-v2
echo "Ligne ajoutée sur la feature" >> notes.txt
git add notes.txt
git commit -m "Add v2 line to notes.txt"

# 7.
git switch main
git merge feature/notes-v2

# 8.
echo "tmp/" >> .gitignore
mkdir tmp && echo "fichier temporaire" > tmp/scratch.txt
git status                  # tmp/ n'apparaît pas

# 9.
git tag -a v0.1.0 -m "Version 0.1.0"

# 10.
git remote add origin https://github.com/<user>/exo-git-1.git
git push -u origin main
git push origin --tags
```

**Points clés à vérifier** :
- Le diff à l'étape 3 est vide dans `git diff --staged` tant que rien n'est ajouté au staging (étape 4 le confirme).
- `.gitignore` n'empêche pas un fichier **déjà suivi** d'apparaître — ici `tmp/scratch.txt` n'a jamais été ajouté, donc il est correctement ignoré dès le départ.
- Un tag n'est jamais poussé par un simple `git push` : il faut `git push origin --tags` (ou `git push origin v0.1.0`).
