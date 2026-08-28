# Niveau 3 — Avancé

## Exercice 3.1 — N+1 et select_related/prefetch_related

Modélise `Category` (`name`) et ajoute un `ForeignKey` `category` sur `Product`. Écris une view qui liste les produits avec le nom de leur catégorie, d'abord **sans** optimisation (observe le nombre de requêtes SQL avec Django Debug Toolbar ou `django.db.connection.queries`), puis corrige avec `select_related`. Compare le nombre de requêtes avant/après.

## Exercice 3.2 — Signal

Écris un signal `post_save` sur `Product` qui affiche en console un message quand `in_stock` passe de `True` à `False` (indice : comparer avec l'état précédent nécessite soit un `pre_save`, soit de stocker l'état initial sur l'instance). Explique en 2-3 lignes le lien avec le principe de découplage des Events Symfony.

## Exercice 3.3 — Manager custom

Écris un manager custom `InStockManager` qui retourne uniquement les produits en stock, disponible via `Product.in_stock_objects.all()`, en conservant le manager par défaut `objects` intact.

## Exercice 3.4 — API REST avec DRF

Installe Django REST Framework. Écris un `ModelSerializer` pour `Product` et un `ModelViewSet`, routé via un `DefaultRouter` sous `/api/products/`. Vérifie que les opérations CRUD (GET liste, GET détail, POST, PUT, DELETE) fonctionnent via un client HTTP (curl, Postman, ou `httpie`).
