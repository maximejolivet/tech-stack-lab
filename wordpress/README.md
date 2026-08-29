# WordPress

## 1. Introduction

WordPress est un CMS (Content Management System) open source écrit en PHP, à l'origine un moteur de blog, devenu au fil du temps un système de gestion de contenu généraliste utilisé par environ 40% des sites web. Ce dossier suppose le PHP natif acquis (voir [`../php/`](../php/)) et les bases SQL (voir [`../mysql/`](../mysql/)), et se concentre sur ce qui est **spécifique à WordPress**.

**Différence fondamentale avec Symfony/Laravel** : ce n'est pas un framework qu'on assemble brique par brique, c'est une **application complète** livrée avec sa propre architecture (thèmes, plugins, schéma de base de données figé) qu'on étend via des **points d'extension** (hooks : actions et filtres) plutôt que de la réécrire.

**À quoi sert-il ?**
- Sites vitrines, blogs, portails de contenu, e-commerce (via l'extension WooCommerce).
- Permettre à des utilisateurs non-techniques de gérer du contenu via une interface d'administration, tout en restant extensible par du code (thèmes, plugins) pour les développeurs.
- De plus en plus utilisé en mode **headless** : WordPress comme back-office de contenu uniquement, exposé via son API REST, consommé par un frontend séparé (voir [`../react/`](../react/), [`../nuxtjs/`](../nuxtjs/)).

**Où se situe-t-il dans une architecture web ?** Historiquement monolithique côté serveur (PHP + MySQL, rendu HTML côté serveur à chaque requête). Peut aussi être découplé (headless) : WordPress ne sert alors que de source de données via son API.

**Avantages** : écosystème de thèmes/plugins immense, admin UI puissante accessible aux non-développeurs, communauté et documentation massives, hébergement supporté partout.
**Limites** : qualité très variable des plugins/thèmes tiers (impact sécurité et performance), architecture historique procédurale avec des hooks globaux (moins "propre" qu'un framework moderne à conteneur de services), surface d'attaque à surveiller en continu (cœur + plugins + thème doivent être tenus à jour).

## 2. Prérequis

- PHP procédural et POO (voir [`../php/`](../php/)) — le cœur de WordPress mélange les deux styles selon l'ancienneté du code.
- Bases SQL (voir [`../mysql/`](../mysql/)) — WordPress stocke tout dans un schéma de tables prédéfini (`wp_posts`, `wp_postmeta`, `wp_options`, `wp_users`...).
- HTML/CSS/JS de base pour le développement de thèmes (voir [`../html/`](../html/), [`../css/`](../css/), [`../javascript/`](../javascript/)).
- Un environnement local avec PHP + MySQL + serveur web (Local, DevKinsta, ou Docker), et [WP-CLI](https://wp-cli.org/) installé.

## 3. Rappel des bases 🟢

### 01 - Structure des dossiers

**Explication** — Une installation WordPress a trois zones principales : `wp-admin/` et `wp-includes/` (cœur, **jamais modifiés directement**), et `wp-content/` (tout ce qu'un développeur ou utilisateur ajoute : `themes/`, `plugins/`, `uploads/`). La configuration (connexion base de données, clés de sécurité) vit dans `wp-config.php` à la racine.

```php
// wp-config.php
define('DB_NAME', 'mon_site');
define('DB_USER', 'root');
define('DB_PASSWORD', 'secret');
define('DB_HOST', 'localhost');
define('WP_DEBUG', true); // à activer en dev uniquement
```

**Bonne pratique** : ne jamais modifier `wp-admin/` ou `wp-includes/` — ces dossiers sont écrasés à chaque mise à jour du cœur. Toute personnalisation passe par `wp-content/themes/` ou `wp-content/plugins/`.

### 02 - Structure minimale d'un thème

**Explication** — Un thème minimal nécessite deux fichiers : `style.css` (avec un en-tête de commentaire obligatoire qui déclare le thème à WordPress) et `index.php` (template de secours utilisé quand aucun template plus spécifique ne correspond).

```css
/* style.css */
/*
Theme Name: Mon Thème
Version: 1.0
*/
```

```php
<?php // index.php
get_header();
if (have_posts()) : while (have_posts()) : the_post(); ?>
    <h1><?php the_title(); ?></h1>
    <div><?php the_content(); ?></div>
<?php endwhile; endif;
get_footer();
```

**Bonne pratique** : découper le thème en templates (`header.php`, `footer.php`, `single.php`, `page.php`) inclus via `get_header()`/`get_footer()` plutôt que de tout dupliquer dans chaque fichier.

### 03 - The Loop

**Explication** — "The Loop" est le mécanisme central d'affichage de contenu : WordPress prépare une requête (`WP_Query`) selon le contexte (page, article, archive...), et `have_posts()`/`the_post()` itèrent sur les résultats en positionnant les fonctions de template (`the_title()`, `the_content()`...) sur l'article courant.

**Erreur fréquente** : appeler `the_title()`/`the_content()` en dehors d'une Loop active — ces fonctions dépendent d'un article "courant" positionné par `the_post()`, sans quoi elles n'affichent rien ou une donnée incohérente.

**Bonne pratique** : après une requête `WP_Query` personnalisée, toujours appeler `wp_reset_postdata()` une fois la boucle terminée, pour ne pas polluer la Loop principale de la page.

### 04 - Posts, Pages et types de contenu

**Explication** — WordPress fournit deux types de contenu natifs : les **Posts** (articles, chronologiques, classables par catégories/tags) et les **Pages** (contenu statique, hiérarchique, sans taxonomie). Au-delà, on peut déclarer ses propres **Custom Post Types** (voir section 4) pour modéliser d'autres entités (produits, événements, témoignages...).

### 05 - Hooks : Actions et Filtres

**Explication** — Le mécanisme d'extension central de WordPress. Une **action** exécute du code à un moment donné, sans rien retourner (effet de bord). Un **filtre** reçoit une valeur, la modifie, et **doit la retourner**.

```php
// Action : exécuter du code à un moment donné
add_action('wp_footer', function () {
    echo '<!-- ajouté en bas de page -->';
});

// Filtre : transformer une valeur existante
add_filter('the_title', function (string $title): string {
    return $title . ' 🚀';
});
```

**Erreur fréquente** : oublier de `return` la valeur dans un callback de filtre — le filtre retournerait alors `null`, effaçant la donnée pour tous les filtres suivants et pour l'affichage final.

**Bonne pratique** : préférer les hooks à la modification directe des fichiers du cœur ou d'un plugin tiers — c'est ce qui garantit que les personnalisations survivent aux mises à jour.

### 06 - Enqueuing des scripts et styles

**Explication** — WordPress fournit un système de dépendances pour charger CSS/JS proprement, plutôt que d'écrire des balises `<script>`/`<link>` en dur dans les templates.

```php
add_action('wp_enqueue_scripts', function () {
    wp_enqueue_style('mon-theme-style', get_stylesheet_uri(), [], '1.0');
    wp_enqueue_script('mon-theme-js', get_template_directory_uri() . '/js/main.js', ['jquery'], '1.0', true);
});
```

**Erreur fréquente** : coder en dur `<script src="...">` dans `header.php` — cela empêche WordPress de gérer les dépendances et le versioning (cache-busting), et peut charger un même script plusieurs fois si un plugin l'enqueue aussi.

**Bonne pratique** : toujours enqueue via `wp_enqueue_scripts` (front) ou `admin_enqueue_scripts` (admin), jamais de balise codée en dur dans un template.

### 07 - Structure minimale d'un plugin

**Explication** — Un plugin est un fichier PHP (ou dossier) avec un en-tête de commentaire similaire à celui d'un thème, placé dans `wp-content/plugins/`. Contrairement au `functions.php` d'un thème, un plugin reste actif même si l'on change de thème.

```php
<?php
/**
 * Plugin Name: Mon Plugin
 * Version: 1.0
 */

add_action('init', function () {
    // logique du plugin
});
```

**Bonne pratique** : toute logique qui n'est pas strictement liée à la présentation visuelle (custom post types, intégrations API, logique métier) va dans un **plugin**, pas dans `functions.php` — sinon elle disparaît si l'utilisateur change de thème.

## 4. Concepts intermédiaires 🟡

- **Custom Post Types et taxonomies personnalisées** : déclarer ses propres entités de contenu et ses propres systèmes de classification.

```php
add_action('init', function () {
    register_post_type('testimonial', [
        'label' => 'Témoignages',
        'public' => true,
        'supports' => ['title', 'editor', 'thumbnail'],
        'show_in_rest' => true, // expose le CPT dans l'API REST native
    ]);
});
```

- **Custom Fields et meta boxes** : stocker des données additionnelles sur un post via `add_post_meta()`/`update_post_meta()`, affichées dans l'admin via `add_meta_box()`. En pratique, la majorité des projets utilisent le plugin [Advanced Custom Fields (ACF)](https://www.advancedcustomfields.com/) plutôt que l'API native, plus verbeuse.
- **WP_Query avancé** : `meta_query` et `tax_query` pour filtrer par champ personnalisé ou taxonomie ; le filtre `pre_get_posts` pour modifier la requête principale d'une page (ex. archive) sans créer de requête secondaire.

```php
$query = new WP_Query([
    'post_type' => 'testimonial',
    'posts_per_page' => 5,
    'meta_query' => [['key' => 'rating', 'value' => 4, 'compare' => '>=']],
]);
```

**Erreur fréquente** : utiliser la fonction dépréciée `query_posts()` pour modifier la Loop principale — elle recalcule toute la requête globale et casse la pagination. Utiliser `pre_get_posts` (pour la requête principale) ou une nouvelle instance `WP_Query` (pour une requête secondaire).

- **Widgets et sidebars** : zones enregistrées (`register_sidebar()`) où l'utilisateur place des blocs de contenu depuis l'admin (`dynamic_sidebar()` pour les afficher dans un template).
- **Menus de navigation** : `register_nav_menus()` déclare des emplacements de menu que l'utilisateur configure dans l'admin, affichés via `wp_nav_menu()`.
- **Shortcodes** : balises `[mon_shortcode]` insérables dans le contenu, résolues en HTML via `add_shortcode()`. De plus en plus remplacés par les **blocs Gutenberg** pour l'édition visuelle (voir section avancée), mais encore très utilisés pour du contenu dynamique simple.
- **API REST native** : chaque installation expose `/wp-json/wp/v2/posts`, `/wp-json/wp/v2/pages`... sans configuration, et les Custom Post Types déclarés avec `show_in_rest: true` en héritent automatiquement. Base du mode headless (voir section avancée).
- **Nonces et sécurité des formulaires** : un nonce (`wp_nonce_field()` côté formulaire, `check_admin_referer()`/`wp_verify_nonce()` côté traitement) protège contre le CSRF — **obligatoire** sur tout formulaire admin qui modifie une donnée.
- **Thèmes enfants (child themes)** : un thème enfant hérite d'un thème parent (`style.css` avec `Template: nom-du-parent`) et ne redéfinit que ce qui change — permet de personnaliser un thème tiers sans perdre ses personnalisations à la prochaine mise à jour du parent.

## 5. Concepts avancés 🟠🔴

- **Sanitization vs escaping — la règle d'or** : *sanitize early, escape late*. On **nettoie** (`sanitize_text_field()`, `sanitize_email()`) une donnée dès qu'elle entre (ex. `$_POST`), et on **échappe** (`esc_html()`, `esc_attr()`, `esc_url()`) une donnée juste avant de l'afficher, quel que soit son parcours entre-temps.

```php
$title = sanitize_text_field($_POST['title'] ?? ''); // à l'entrée
// ... plus tard, à l'affichage :
echo '<h2>' . esc_html($title) . '</h2>'; // juste avant le echo
```

**Erreur fréquente** : sanitizer à l'entrée et considérer que c'est suffisant pour l'affichage — une donnée sanitizée n'est pas forcément sûre dans **tous** les contextes de sortie (HTML, attribut, URL, JS) ; chaque contexte a sa propre fonction d'échappement à appliquer au moment de l'affichage.

- **Requêtes directes via `$wpdb`** : quand `WP_Query` ne suffit pas (agrégations, jointures complexes), on interroge la base directement — **toujours** avec `prepare()` pour éviter l'injection SQL.

```php
global $wpdb;
$results = $wpdb->get_results(
    $wpdb->prepare("SELECT * FROM {$wpdb->prefix}posts WHERE post_author = %d", $userId)
);
```

- **Transients API et Object Cache** : `set_transient()`/`get_transient()` mettent en cache une donnée coûteuse à recalculer (résultat d'API externe, requête lourde) avec une expiration. En production, un plugin d'Object Cache persistant (Redis — voir [`../redis/`](../redis/)) transforme ces transients en cache réellement partagé entre requêtes, au lieu de retomber sur la table `wp_options`.
- **Performance** : cache de page (plugin dédié ou reverse proxy), CDN pour les assets/médias, limiter le nombre de plugins actifs (chaque plugin ajoute des hooks exécutés à chaque requête), profiler les requêtes lentes avec le plugin [Query Monitor](https://querymonitor.com/).
- **Développement de blocs Gutenberg** : l'éditeur de contenu moderne (Gutenberg) est basé sur React ; un bloc se déclare via un fichier `block.json` (métadonnées) et un composant JS (`registerBlockType`) — voir [`../react/`](../react/) pour les bases nécessaires.
- **WordPress headless** : utiliser WordPress uniquement comme back-office de contenu (API REST native ou [WPGraphQL](https://www.wpgraphql.com/)), consommé par un frontend séparé (Next.js, Nuxt — voir [`../nuxtjs/`](../nuxtjs/)) ; gagne en flexibilité front et en sécurité (surface d'attaque PHP non exposée publiquement) mais perd les fonctionnalités admin intégrées (prévisualisation, thèmes visuels).
- **Multisite** : une seule installation WordPress peut piloter un réseau de sites partageant le même cœur et les mêmes plugins (utile pour des déclinaisons multi-langues/multi-marques), avec ses propres contraintes de configuration (`wp-config.php`, table `wp_blogs`).
- **WP-Cron** : contrairement à un cron système classique, WP-Cron se déclenche uniquement lors d'une visite sur le site (pas d'exécution garantie sur un site à faible trafic). En production, il est recommandé de désactiver le déclenchement automatique (`define('DISABLE_WP_CRON', true);`) et de le remplacer par un vrai cron serveur qui appelle `wp-cron.php` à intervalle régulier.
- **Architecture de plugin avancée** : sur un plugin conséquent, structurer le code en classes avec autoload Composer (`PSR-4`) plutôt qu'un unique fichier procédural avec des fonctions globales — évite les collisions de noms et facilite les tests (voir [`../design-patterns/`](../design-patterns/), [`../php/`](../php/)).

## 6. Commandes / syntaxe à connaître

```bash
wp core download                     # télécharger le cœur WordPress
wp core install --url=... --title=... --admin_user=... --admin_email=...
wp plugin install <slug> --activate  # installer et activer un plugin
wp theme activate <slug>             # activer un thème
wp db export backup.sql              # exporter la base
wp db import backup.sql              # importer la base
wp search-replace 'ancien.test' 'nouveau.test'  # remplacer une URL dans toute la base
wp user create alice alice@test.com --role=editor
wp cron event list                   # lister les tâches WP-Cron programmées
wp cron event run --due-now          # forcer l'exécution des tâches dues
```

```php
add_action('hook_name', callback, priority: 10, accepted_args: 1);
add_filter('hook_name', callback, priority: 10, accepted_args: 1);
register_post_type('slug', ['public' => true, 'supports' => [...]]);
new WP_Query(['post_type' => 'post', 'posts_per_page' => 10]);
wp_enqueue_script('handle', $src, $deps, $version, in_footer: true);
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Plugin "Témoignages"**

Construire un plugin WordPress complet (pas un thème) avec :
- Un Custom Post Type `testimonial` (non hiérarchique, support `title` + `editor`), exposé dans l'API REST (`show_in_rest`).
- Un champ personnalisé `rating` (note de 1 à 5) ajouté via une meta box native, protégée par un nonce et sauvegardée avec `update_post_meta()` après sanitization.
- Un shortcode `[testimonials count="3"]` qui affiche les X derniers témoignages via `WP_Query`, avec échappement correct (`esc_html`) de chaque donnée affichée.
- Un endpoint REST personnalisé `GET /wp-json/mon-plugin/v1/testimonials` (`register_rest_route`) qui retourne les témoignages en JSON, pour un usage headless.
- Un enqueue conditionnel du CSS du plugin, chargé uniquement sur les pages où le shortcode est présent.
- Bonus : mettre en cache le rendu du shortcode dans un transient (invalidé quand un témoignage est publié/modifié via le hook `save_post`), et programmer une tâche WP-Cron qui purge les témoignages non publiés depuis plus de 30 jours.

## Checklist

- [ ] Comprendre les fondamentaux (structure, thèmes, The Loop, hooks actions/filtres)
- [ ] Savoir installer WordPress en local et créer un thème minimal
- [ ] Maîtriser la syntaxe principale (`add_action`/`add_filter`, enqueue, `register_post_type`)
- [ ] Comprendre les concepts importants (Custom Post Types, `WP_Query`, API REST native, nonces)
- [ ] Savoir debugger (`WP_DEBUG`, Query Monitor, logs)
- [ ] Connaître les bonnes pratiques (sanitize/escape, éviter `query_posts()`, plugin vs thème)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (headless, blocs Gutenberg, sécurité avancée, multisite)

## 10. Ressources

- [Developer Resources officiel](https://developer.wordpress.org/) — point d'entrée vers tous les handbooks.
- [Plugin Handbook](https://developer.wordpress.org/plugins/) et [Theme Handbook](https://developer.wordpress.org/themes/) — références pour le développement de thèmes/plugins.
- [Block Editor Handbook](https://developer.wordpress.org/block-editor/) — développement de blocs Gutenberg (React).
- [WP-CLI](https://wp-cli.org/) — documentation de l'outil en ligne de commande.
- [Query Monitor](https://querymonitor.com/) — plugin de debug/profiling indispensable en développement.
- Il n'existe pas de roadmap.sh dédiée à WordPress à ce jour ; voir [roadmap.sh — PHP](https://roadmap.sh/php) pour le contexte langage plus large.
