# Niveau 1 — Bases

## Exercice 1.1 — Premier projet

Crée un projet Django `myshop` et une app `catalog`. Déclare l'app dans `INSTALLED_APPS`. Lance le serveur de dev et vérifie que la page d'accueil par défaut s'affiche.

## Exercice 1.2 — View minimale

Dans `catalog/views.py`, écris une view `hello` qui retourne un `HttpResponse("Bienvenue dans la boutique")`. Déclare la route `""` dans `catalog/urls.py`, inclus-la dans le `urls.py` racine, et vérifie dans le navigateur.

## Exercice 1.3 — Model et migration

Modélise `Product` (`name` CharField, `price` DecimalField, `in_stock` BooleanField par défaut `True`). Génère et applique la migration. Vérifie la table créée avec `python manage.py sqlmigrate`.

## Exercice 1.4 — Template avec boucle

Crée une view `product_list` qui récupère tous les `Product` et les passe à un template `catalog/list.html`. Le template doit afficher chaque produit dans une `<li>` avec son nom et son prix, via `{% for %}`.

## Exercice 1.5 — Admin

Enregistre `Product` dans `admin.py` avec `list_display = ["name", "price", "in_stock"]`. Crée un superuser et vérifie que tu peux ajouter/modifier un produit depuis `/admin/`.
