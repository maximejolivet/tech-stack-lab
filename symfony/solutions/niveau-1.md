# Solutions — Niveau 1

## Exercice 1

```php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class HelloController
{
    #[Route('/hello/{name}', name: 'hello', requirements: ['name' => '[a-zA-Z]+'])]
    public function index(string $name): Response
    {
        return new Response("Bonjour, {$name} !");
    }
}
```

## Exercice 2

```php
// src/Controller/ArticleController.php
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class ArticleController extends AbstractController
{
    #[Route('/articles', name: 'article_list')]
    public function list(): Response
    {
        return $this->render('article/list.html.twig', [
            'titles' => ['PHP 8.3', 'Symfony 7', 'Doctrine ORM'],
        ]);
    }
}
```

```twig
{# templates/article/list.html.twig #}
{% extends 'base.html.twig' %}
{% block body %}
  <ul>
    {% for title in titles %}
      <li>{{ title }}</li>
    {% endfor %}
  </ul>
{% endblock %}
```

## Exercice 3

- `bin/` : point d'entrée CLI (`bin/console`).
- `config/` : configuration YAML (routes, services, packages).
- `public/` : document root du serveur web, seul répertoire exposé (contient `index.php`).
- `src/` : code applicatif (contrôleurs, entités, services, repositories).
- `templates/` : vues Twig.
- `tests/` : tests PHPUnit.
- `var/` : cache et logs générés (non versionné).
- `vendor/` : dépendances Composer (non versionné).

## Exercice 4

`.env` contient les valeurs par défaut/de démo des variables d'environnement et sert de documentation de ce qui est attendu — il est versionné. `.env.local` contient les valeurs réelles locales (secrets, credentials de base de données locale) et surcharge `.env` ; il est ignoré par Git (`.gitignore`) car spécifique à chaque environnement/développeur et potentiellement sensible.
