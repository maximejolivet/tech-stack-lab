# Exercices WordPress — Niveau 1 (Bases)

## Exercice 1 — En-tête de thème

Écris l'en-tête de commentaire minimal d'un fichier `style.css` pour un thème nommé "Blog Perso", version 1.0.

## Exercice 2 — The Loop

Écris le contenu de `index.php` qui affiche, pour chaque article de la Loop, son titre dans un `<h2>` et son extrait (`the_excerpt()`) dans un `<p>`.

## Exercice 3 — Action simple

Utilise `add_action` pour afficher le texte `<!-- Site propulsé par WordPress -->` juste avant la fermeture du `</body>` (indice : quel hook s'exécute à cet endroit ?).

## Exercice 4 — Filtre simple

Utilise `add_filter` pour que tous les titres d'articles (`the_title`) soient automatiquement affichés en majuscules.

## Exercice 5 — Enqueue correct vs incorrect

Voici un extrait de `header.php` :

```php
<head>
    <script src="<?php echo get_template_directory_uri(); ?>/js/main.js"></script>
</head>
```

Explique en une phrase pourquoi ce n'est pas une bonne pratique, puis réécris l'équivalent correct avec `wp_enqueue_script` accroché au bon hook.
