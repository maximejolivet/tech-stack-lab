# Solutions — Niveau 3 (Avancé)

## Exercice 1

```php
global $wpdb;

$count = $wpdb->get_var($wpdb->prepare(
    "SELECT COUNT(*) FROM {$wpdb->posts} WHERE post_type = %s AND post_author = %d AND post_status = %s",
    'event',
    $userId,
    'publish'
));
```

## Exercice 2

```php
add_shortcode('latest_events', function ($atts) {
    $atts = shortcode_atts(['count' => 5], $atts);
    $cacheKey = 'latest_events_' . (int) $atts['count'];

    $cached = get_transient($cacheKey);
    if ($cached !== false) {
        return $cached;
    }

    $query = new WP_Query([
        'post_type' => 'event',
        'posts_per_page' => (int) $atts['count'],
        'orderby' => 'date',
        'order' => 'DESC',
    ]);

    $output = '<ul>';
    while ($query->have_posts()) {
        $query->the_post();
        $date = esc_html(get_post_meta(get_the_ID(), 'event_date', true));
        $output .= '<li>' . esc_html(get_the_title()) . ' — ' . $date . '</li>';
    }
    wp_reset_postdata();
    $output .= '</ul>';

    set_transient($cacheKey, $output, HOUR_IN_SECONDS);

    return $output;
});
```

## Exercice 3

```php
add_action('save_post_event', function ($postId) {
    // Supprime tous les transients de comptage connus ; en production,
    // on maintiendrait une liste des clés utilisées pour cibler la suppression.
    global $wpdb;
    $wpdb->query(
        "DELETE FROM {$wpdb->options} WHERE option_name LIKE '_transient_latest_events_%'"
    );
});
```

## Exercice 4

```php
add_action('rest_api_init', function () {
    register_rest_route('mon-plugin/v1', '/events', [
        'methods' => 'GET',
        'callback' => function () {
            $query = new WP_Query([
                'post_type' => 'event',
                'posts_per_page' => 20,
                'meta_key' => 'event_date',
                'orderby' => 'meta_value',
                'order' => 'ASC',
                'meta_query' => [
                    ['key' => 'event_date', 'value' => date('Y-m-d'), 'compare' => '>=', 'type' => 'DATE'],
                ],
            ]);

            $events = array_map(function ($post) {
                return [
                    'title' => get_the_title($post),
                    'date' => get_post_meta($post->ID, 'event_date', true),
                    'link' => get_permalink($post),
                ];
            }, $query->posts);

            return rest_ensure_response($events);
        },
        'permission_callback' => '__return_true',
    ]);
});
```

## Exercice 5

```php
add_action('purge_old_events', function () {
    global $wpdb;

    $cutoff = date('Y-m-d', strtotime('-90 days'));
    $ids = $wpdb->get_col($wpdb->prepare(
        "SELECT p.ID FROM {$wpdb->posts} p
         INNER JOIN {$wpdb->postmeta} pm ON pm.post_id = p.ID
         WHERE p.post_type = %s AND pm.meta_key = 'event_date' AND pm.meta_value < %s",
        'event',
        $cutoff
    ));

    foreach ($ids as $id) {
        wp_delete_post($id, true);
    }
});

if (!wp_next_scheduled('purge_old_events')) {
    wp_schedule_event(time(), 'daily', 'purge_old_events');
}
```

WP-Cron se déclenche uniquement lors d'une visite sur le site (ce n'est pas un vrai cron système) : sur un site à très faible trafic, la tâche peut ne jamais s'exécuter au moment prévu, voire pas du tout pendant des jours. En production, la recommandation est de désactiver le déclenchement automatique (`define('DISABLE_WP_CRON', true);` dans `wp-config.php`) et de configurer un vrai cron serveur qui appelle `wp-cron.php` à intervalle régulier.
