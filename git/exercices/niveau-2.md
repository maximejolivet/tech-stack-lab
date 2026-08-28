# Git — Exercices Niveau 2 (Intermédiaire)

Objectif : combiner plusieurs notions (rebase, stash, cherry-pick, reset/revert, résolution de conflits).

1. Dans un dépôt existant, fais 3 commits "sales" sur une branche `feature/x` (`wip 1`, `wip 2`, `fix typo`). Utilise `git rebase -i` pour les fusionner (squash) en un seul commit avec un message propre.
2. Crée deux branches à partir de `main` qui modifient toutes les deux la **même ligne** d'un même fichier. Merge la première sans problème, puis merge la seconde : résous le conflit manuellement (sans outil graphique) et vérifie que le résultat final contient bien le sens des deux modifications.
3. Commence à modifier un fichier, puis (sans committer) mets ces modifications de côté avec `git stash`. Change de branche, fais un commit différent, reviens sur la branche d'origine et réapplique le stash.
4. Sur une branche `main` avec au moins 5 commits, identifie le hash d'un commit "au milieu" et applique-le (cherry-pick) sur une branche `hotfix` créée à partir d'un commit plus ancien.
5. Fais un commit, puis annule-le de deux façons différentes sur deux branches séparées : une fois avec `git reset --soft`, une fois avec `git revert`. Explique (en commentaire dans un fichier `reponses.md`) la différence observée dans `git log` entre les deux approches.
6. Configure un alias Git personnel `git lg` qui affiche un log condensé avec graphe (`--oneline --graph --all --decorate`).
7. Fais volontairement un `git reset --hard` qui supprime un commit contenant du travail important, puis récupère ce commit uniquement via `git reflog` (sans undo de ton éditeur, sans backup externe).
