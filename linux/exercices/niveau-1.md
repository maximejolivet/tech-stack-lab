# Exercices Linux — Niveau 1 (Bases)

## Exercice 1 — Navigation

Depuis ton répertoire personnel, déplace-toi vers `/var/log`, affiche le chemin absolu courant, puis reviens directement au répertoire précédent en une seule commande.

## Exercice 2 — Permissions

Crée un fichier `deploy.sh`, rends-le exécutable uniquement pour le propriétaire et le groupe (pas pour les autres), puis affiche le résultat avec `ls -l` pour vérifier.

## Exercice 3 — Processus

Liste tous les processus dont le nom contient "ssh", puis explique la différence entre envoyer un `kill -15` et un `kill -9` à un processus.

## Exercice 4 — Pipes et redirections

Étant donné un fichier `access.log`, écris une seule ligne de commande qui compte le nombre de lignes contenant le mot "ERROR", puis une autre commande qui ajoute (sans écraser) la ligne "Fin de rapport" à la fin de ce fichier.

## Exercice 5 — Paquets et utilisateurs

Sur une distribution Debian/Ubuntu, écris les commandes pour mettre à jour la liste des paquets, installer `htop`, créer un utilisateur `bob` avec un répertoire personnel, et l'ajouter au groupe `sudo`.
