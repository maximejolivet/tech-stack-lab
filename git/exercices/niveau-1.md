# Git — Exercices Niveau 1 (Bases)

Objectif : vérifier la maîtrise du cycle de base (init, add, commit, branch, merge, remote).

1. Crée un nouveau dépôt Git dans un dossier `exo-git-1`. Vérifie qu'il est bien initialisé.
2. Crée un fichier `notes.txt` contenant une ligne de texte. Ajoute-le au staging puis committe-le avec un message clair.
3. Modifie `notes.txt` (ajoute une ligne), puis affiche le diff **avant** de l'ajouter au staging.
4. Ajoute la modification au staging, affiche le diff **staged**, puis committe.
5. Affiche l'historique des commits en une ligne par commit (`--oneline`).
6. Crée une branche `feature/notes-v2`, bascule dessus, ajoute une ligne dans `notes.txt`, committe.
7. Reviens sur la branche principale et merge `feature/notes-v2` dedans.
8. Crée un fichier `.gitignore` qui exclut un dossier `tmp/`. Vérifie qu'un fichier créé dans `tmp/` n'apparaît pas dans `git status`.
9. Crée un tag annoté `v0.1.0` sur le dernier commit.
10. (Si tu as un compte GitHub) crée un dépôt distant vide, ajoute-le comme remote `origin`, et pousse ta branche principale + les tags.
