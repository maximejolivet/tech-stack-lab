# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```php
add_action('init', function () {
    register_post_type('event', [
        'label' => 'Événements',
        'public' => true,
        'supports' => ['title', 'editor'],
        'show_in_rest' => true,
    ]);
});
```

## Exercice 2

```php
$query = new WP_Query([
    'post_type' => 'event',
    'posts_per_page' => 10,
    'meta_key' => 'event_date',
    'orderby' => 'meta_value',
    'order' => 'ASC',
    'meta_query' => [
        ['key' => 'event_date', 'value' => date('Y-m-d'), 'compare' => '>=', 'type' => 'DATE'],
    ],
]);
```

## Exercice 3

```php
add_shortcode('latest_events', function ($atts) {
    $atts = shortcode_atts(['count' => 5], $atts);

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

    return $output;
});
```

## Exercice 4

```php
// Affichage du champ dans la meta box
add_action('add_meta_boxes', function () {
    add_meta_box('event_location_box', 'Lieu', function ($post) {
        wp_nonce_field('save_event_location', 'event_location_nonce');
        $value = get_post_meta($post->ID, 'event_location', true);
        echo '<input type="text" name="event_location" value="' . esc_attr($value) . '">';
    }, 'event');
});

// Sauvegarde sécurisée
add_action('save_post', function ($postId) {
    if (!isset($_POST['event_location_nonce']) ||
        !wp_verify_nonce($_POST['event_location_nonce'], 'save_event_location')) {
        return;
    }

    if (isset($_POST['event_location'])) {
        update_post_meta($postId, 'event_location', sanitize_text_field($_POST['event_location']));
    }
});
```

## Exercice 5

```php
add_action('pre_get_posts', function (WP_Query $query) {
    if (!is_admin() && $query->is_main_query() && is_post_type_archive('event')) {
        $query->set('meta_key', 'event_date');
        $query->set('orderby', 'meta_value');
        $query->set('order', 'ASC');
    }
});
```
