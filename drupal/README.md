# Drupal

## 1. Introduction

Drupal est un CMS PHP open source plus structurant et orienté entreprise que WordPress (voir [`../wordpress/`](../wordpress/)). Depuis Drupal 8, le cœur est construit sur des composants [Symfony](../symfony/) (routing, dependency injection, événements, HttpFoundation) — une base de code objet moderne plutôt que le style historiquement procédural de WordPress. Ce dossier suppose le PHP natif acquis (voir [`../php/`](../php/)) et les bases Symfony utiles pour comprendre l'architecture sous-jacente.

**À quoi sert-il ?**
- Sites complexes à forte volumétrie de contenu, structures de données très hétérogènes, exigences de gouvernance (portails institutionnels, universités, gouvernements, grands médias).
- Modéliser du contenu très structuré (types de contenu personnalisés avec de nombreux champs typés) sans écrire de code, via l'admin.

**Où se situe-t-il dans une architecture web ?** Full-stack monolithique (PHP + base de données, rendu Twig côté serveur) comme WordPress, mais avec une architecture orientée objet (Plugins, Services, Événements) plus proche d'un framework comme Symfony. Peut aussi fonctionner en mode **découplé (headless)** via son API JSON:API/GraphQL native.

**Avantages** : modèle de contenu très riche et flexible sans coder (Content Types, Fields, Views), système de permissions granulaire nativement robuste, architecture orientée objet testable, forte adoption dans le secteur public/enterprise.
**Limites** : courbe d'apprentissage nettement plus raide que WordPress, écosystème de modules moins large, plus lourd à héberger et à maintenir pour un simple site vitrine — souvent disproportionné si les besoins sont simples.

## 2. Prérequis

- PHP orienté objet solide : interfaces, injection de dépendances, namespaces (voir [`../php/`](../php/)).
- Notions Symfony de base (voir [`../symfony/`](../symfony/)) : Drupal réutilise directement son conteneur de services, son routing YAML et son système d'événements.
- Bases SQL (voir [`../mysql/`](../mysql/)).
- [Composer](https://getcomposer.org/) et [Drush](https://www.drush.org/) installés.

## 3. Rappel des bases 🟢

### 01 - Installation et structure d'un projet

**Explication** — Un projet Drupal moderne se crée via Composer (`composer create-project drupal/recommended-project`). Structure clé : `web/` (racine servie, contient `core/`, `modules/`, `themes/`, `sites/default/settings.php`), `web/modules/custom/` (code métier), `web/themes/custom/` (thème).

```bash
composer create-project drupal/recommended-project mon-site
cd mon-site
composer require drush/drush
./vendor/bin/drush site:install
```

**Bonne pratique** : ne jamais modifier `web/core/` directement — comme `wp-admin/` chez WordPress, ce dossier est écrasé à chaque mise à jour. Toute personnalisation va dans `web/modules/custom/` ou `web/themes/custom/`.

### 02 - Types de contenu et champs

**Explication** — Un "Content Type" (ex. `article`, `page`) définit une structure de contenu avec des champs typés (texte, image, référence à une entité, date...), configurable entièrement depuis l'admin, sans code.

**Différence avec WordPress** : là où WordPress ajoute des Custom Fields au cas par cas (souvent via ACF), Drupal a un système de champs natif et unifié — chaque type de contenu, utilisateur, ou taxonomie peut recevoir n'importe quel champ du même système.

### 03 - Entités et bundles

**Explication** — Une **Entité** est l'unité de données fondamentale de Drupal (Node, User, Taxonomy Term, Comment, Media...). Un **bundle** est une déclinaison typée d'une entité (ex. les Content Types sont les bundles de l'entité `node`).

```php
// Charger et manipuler une entité en code
$node = \Drupal\node\Entity\Node::load($nid);
$title = $node->getTitle();
$node->set('field_summary', 'Résumé mis à jour');
$node->save();
```

**Bonne pratique** : toujours passer par l'API Entité (`Node::load`, `->save()`) plutôt que d'écrire des requêtes SQL directes — elle déclenche les hooks/événements associés (validation, cache, indexation) que des requêtes brutes contourneraient.

### 04 - Modules : structure minimale

**Explication** — L'équivalent des plugins WordPress. Un module minimal nécessite un fichier `mon_module.info.yml` (métadonnées) dans `web/modules/custom/mon_module/`.

```yaml
# mon_module.info.yml
name: 'Mon Module'
type: module
description: 'Un module personnalisé.'
core_version_requirement: ^10
package: Custom
```

**Bonne pratique** : un module = une responsabilité fonctionnelle claire ; éviter le "module fourre-tout" qui mélange plusieurs préoccupations non liées.

### 05 - Hooks (procédural) et Événements (orienté objet)

**Explication** — Drupal conserve un système de hooks procéduraux hérité (fonctions nommées `mon_module_NOM_DU_HOOK()`), tout en introduisant progressivement des **événements** Symfony orientés objet pour les cas structurants (routing, envoi de réponse).

```php
// mon_module.module — hook procédural classique
function mon_module_node_presave(\Drupal\node\NodeInterface $node) {
    if ($node->bundle() === 'article') {
        $node->set('field_slug', \Drupal\Component\Utility\Html::getUniqueId($node->getTitle()));
    }
}
```

**Erreur fréquente** : oublier de vider le cache de découverte des hooks/routes (`drush cache:rebuild`) après avoir ajouté ou modifié un hook — Drupal met en cache agressivement sa configuration et ses définitions de code.

### 06 - Twig (templating)

**Explication** — Comme Symfony, Drupal utilise Twig pour ses templates de thème (`node.html.twig`, `page.html.twig`), avec échappement automatique des variables affichées.

```twig
{# templates/node--article.html.twig #}
<article>
    <h2>{{ label }}</h2>
    <div>{{ content.field_summary }}</div>
</article>
```

**Bonne pratique** : nommer les templates selon le système de suggestions de Drupal (`node--article.html.twig` surcharge `node.html.twig` uniquement pour le bundle `article`) plutôt que de mettre des conditions `{% if %}` dans un template générique.

### 07 - Configuration Management

**Explication** — Toute la configuration du site (types de contenu, champs, vues, permissions) est exportable en fichiers YAML versionnables (`drush config:export`), et ré-importable sur un autre environnement — contrairement à WordPress où la configuration reste en base de données sans mécanisme natif de synchronisation entre environnements.

```bash
drush config:export    # exporte la config active vers config/sync/*.yml
drush config:import    # importe les fichiers YAML dans la base cible
```

**Bonne pratique** : versionner le dossier `config/sync/` dans Git — c'est ce qui permet de déployer une évolution de configuration (nouveau champ, nouvelle vue) de façon reproductible entre dev, staging et prod.

## 4. Concepts intermédiaires 🟡

- **Views** : constructeur de requêtes visuel dans l'admin pour générer des listes de contenu (pages, blocs, flux RSS) sans code — équivalent no-code d'une `WP_Query` avancée côté WordPress, mais bien plus puissant (jointures, agrégations, exposition de filtres à l'utilisateur).
- **Plugin API** : système d'extension orienté objet où chaque "type" de plugin (Block, Field Formatter, Field Widget...) implémente une interface commune et se déclare via une annotation ou un attribut PHP.

```php
/**
 * @Block(
 *   id = "hello_block",
 *   admin_label = @Translation("Hello Block")
 * )
 */
class HelloBlock extends BlockBase {
    public function build(): array {
        return ['#markup' => $this->t('Bonjour !')];
    }
}
```

- **Services et injection de dépendances** : comme en Symfony, la logique réutilisable se déclare en service (`mon_module.services.yml`) et s'injecte plutôt que d'appeler des fonctions globales statiques.
- **Routing** : les routes se déclarent en YAML (`mon_module.routing.yml`), avec un contrôleur associé, exactement comme dans un projet Symfony pur.

```yaml
# mon_module.routing.yml
mon_module.hello:
  path: '/hello'
  defaults:
    _controller: '\Drupal\mon_module\Controller\HelloController::hello'
  requirements:
    _permission: 'access content'
```

- **Système de permissions et rôles** : granularité fine par rôle et par entité (ex. "modifier le contenu d'autrui de type article"), configurable sans code — un cran au-dessus du système de rôles fixes de WordPress.
- **Taxonomies** : système de classification générique (comme les catégories/tags WordPress mais applicable à n'importe quelle entité, avec vocabulaires personnalisés et hiérarchie).
- **Media API** : gestion centralisée des médias (images, vidéos, documents) comme des entités à part entière, réutilisables et référençables depuis n'importe quel champ.

## 5. Concepts avancés 🟠🔴

- **Système de cache (Cache Tags, Cache Contexts)** : chaque portion de rendu (une Vue, un bloc, une page) est mise en cache avec des **tags** décrivant les données dont elle dépend (`node:42`) — quand cette donnée change, seuls les caches associés à ce tag sont invalidés, sans vider tout le cache du site. Les **Cache Contexts** (utilisateur, langue, rôle) déterminent les variations à mettre en cache séparément.
- **BigPipe** : technique de rendu qui envoie immédiatement le HTML "cacheable" d'une page (structure commune) puis streame les portions personnalisées (ex. panier, notifications utilisateur) une fois calculées — évite qu'une seule zone dynamique empêche la mise en cache de toute la page.
- **Migrate API** : framework robuste de migration de données (depuis une autre base, un CSV, un ancien CMS) basé sur des plugins source/process/destination déclarés en YAML — nettement plus structuré que les scripts d'import ad hoc courants sous WordPress.
- **Drupal découplé (headless/decoupled)** : exposer le contenu via JSON:API (activé nativement dans le cœur) ou GraphQL, consommé par un frontend séparé (voir [`../nuxtjs/`](../nuxtjs/), [`../react/`](../react/)) — Drupal reste alors responsable de la modélisation de contenu et des permissions, le rendu est délégué au frontend.
- **Sécurité** : `Xss::filter()`/`Html::escape()` pour l'échappement manuel quand Twig ne suffit pas, permissions vérifiées via `AccessResult` plutôt que des conditions `if` ad hoc éparpillées, mises à jour de sécurité du cœur et des modules contributed suivies via `drush pm:security` — voir [`../security/`](../security/).
- **Multisite et Domain Access** : plusieurs sites peuvent partager un même code base Drupal (`sites/site1`, `sites/site2`), ou un seul site peut servir plusieurs domaines avec du contenu partiellement partagé via le module contributed Domain Access.
- **Architecture de module avancée** : découpage en Services (logique), Plugins (extension typée), Event Subscribers (réaction à un événement Symfony/Drupal), et Entities personnalisées (`ContentEntityBase`) pour modéliser des données qui ne s'intègrent pas naturellement dans le modèle Node — voir [`../design-patterns/`](../design-patterns/) pour les principes SOLID sous-jacents.

## 6. Commandes / syntaxe à connaître

```bash
composer create-project drupal/recommended-project mon-site   # nouveau projet
drush site:install                                             # installer Drupal
drush generate module                                          # scaffolder un module
drush cache:rebuild                                             # vider/reconstruire tous les caches
drush config:export / config:import                             # synchroniser la configuration
drush updatedb                                                   # jouer les mises à jour de schéma (hook_update_N)
drush user:login                                                 # obtenir un lien de connexion admin
drush pm:security                                                 # lister les mises à jour de sécurité disponibles
drush ws --tail                                                   # suivre les logs (watchdog) en direct
```

```php
$node = \Drupal\node\Entity\Node::load($nid);          // charger une entité
\Drupal::entityTypeManager()->getStorage('node');       // API de stockage générique
\Drupal::service('mon_module.mon_service');              // récupérer un service du conteneur
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Module "Annuaire d'événements"**

Construire un module Drupal custom avec :
- Un Content Type `event` avec des champs `field_date` (date), `field_location` (texte) et `field_summary` (texte long).
- Une Vue exposée listant les événements à venir, triée par date, avec un filtre exposé par lieu.
- Un bloc personnalisé (Plugin Block) affichant les 3 prochains événements dans la sidebar.
- Un contrôleur avec une route custom (`/evenements/a-venir`) qui retourne cette même liste en JSON, pour un usage headless.
- Un `hook_node_presave` qui calcule et stocke automatiquement un slug lisible à partir du titre.
- Bonus : exporter la configuration du Content Type et de la Vue en YAML versionné (`config/sync/`), et écrire un plugin Migrate qui importe des événements depuis un CSV.

## Checklist

- [ ] Comprendre les fondamentaux (entités, bundles, modules, hooks, Twig)
- [ ] Savoir installer Drupal via Composer et créer un module minimal
- [ ] Maîtriser la syntaxe principale (API Entité, routing YAML, Plugin API)
- [ ] Comprendre les concepts importants (Views, Services/DI, permissions, Media)
- [ ] Savoir debugger (`drush ws`, `cache:rebuild`, Devel module)
- [ ] Connaître les bonnes pratiques (Config Management versionné, API Entité plutôt que SQL brut)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Cache Tags/Contexts, BigPipe, Migrate API, découplé)

## 10. Ressources

- [Drupal.org — Documentation officielle](https://www.drupal.org/docs) — référence complète, toujours vérifier la version documentée.
- [Drush](https://www.drush.org/) — documentation de l'outil en ligne de commande.
- [Drupal API Reference](https://api.drupal.org/) — référence des classes et interfaces du cœur.
- [Examples for Developers](https://www.drupal.org/project/examples) — module contributed avec des exemples de code idiomatiques par sous-système.
- Il n'existe pas de roadmap.sh dédiée à Drupal à ce jour ; voir [roadmap.sh — PHP](https://roadmap.sh/php) pour le contexte langage plus large, et [`../symfony/`](../symfony/) pour les composants partagés.
