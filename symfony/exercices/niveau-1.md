# Exercices — Niveau 1 (Bases)

## Exercice 1 — Route paramétrée

Crée une route `/hello/{name}` sur `HelloController::index(string $name)` qui retourne une `Response` texte `"Bonjour, {name} !"`. Contrainds `{name}` pour n'accepter que des lettres (`[a-zA-Z]+`).

## Exercice 2 — Vue Twig

Crée un contrôleur `ArticleController::list()` qui passe un tableau de 3 titres d'articles (strings en dur) à un template `article/list.html.twig` affichant une `<ul>` avec un `<li>` par titre.

## Exercice 3 — Structure de projet

Sans regarder la doc, liste de mémoire les répertoires principaux d'un projet Symfony (`bin/`, `config/`, ...) et explique en une phrase le rôle de chacun. Vérifie ensuite avec le README.

## Exercice 4 — Configuration

Explique la différence entre `.env` et `.env.local`, et pourquoi seul le premier est versionné dans Git.
