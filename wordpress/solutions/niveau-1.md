# Solutions — Niveau 1 (Bases)

## Exercice 1

```css
/*
Theme Name: Blog Perso
Version: 1.0
*/
```

## Exercice 2

```php
<?php if (have_posts()) : while (have_posts()) : the_post(); ?>
    <h2><?php the_title(); ?></h2>
    <p><?php the_excerpt(); ?></p>
<?php endwhile; endif; ?>
```

## Exercice 3

Le hook `wp_footer` s'exécute juste avant `</body>`.

```php
add_action('wp_footer', function () {
    echo '<!-- Site propulsé par WordPress -->';
});
```

## Exercice 4

```php
add_filter('the_title', function (string $title): string {
    return strtoupper($title);
});
```

## Exercice 5

Coder `<script src="...">` en dur empêche WordPress de gérer les dépendances et le versioning (cache-busting), et peut provoquer un chargement en double si un plugin enqueue déjà ce même script.

```php
add_action('wp_enqueue_scripts', function () {
    wp_enqueue_script('mon-theme-main', get_template_directory_uri() . '/js/main.js', [], '1.0', true);
});
```
