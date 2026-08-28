# Niveau 2 — Intermédiaire

## Exercice 2.1 — ModelForm

Crée un `ModelForm` pour `Product` (champs `name`, `price`, `in_stock`). Écris une view `create_product` qui affiche le formulaire en GET et le sauvegarde en POST après validation (`form.is_valid()`).

## Exercice 2.2 — QuerySets

À partir du modèle `Product` (niveau 1), écris une vue qui retourne, via des QuerySets chaînés (pas de boucle Python manuelle pour filtrer) :
1. Les produits en stock, triés par prix croissant.
2. Le nombre de produits hors stock (`.count()`).
3. Les 5 produits les plus chers (`order_by("-price")[:5]`).

## Exercice 2.3 — Class-Based Views

Réécris la view `product_list` du niveau 1 en `ListView` générique, avec `queryset` filtré sur `in_stock=True`. Ajoute une `DetailView` pour afficher un produit par son id.

## Exercice 2.4 — Middleware custom

Écris un middleware qui ajoute un header `X-Powered-By: Django-Lab` à chaque réponse. Déclare-le dans `MIDDLEWARE` et vérifie sa présence dans les headers de réponse (outils dev du navigateur).

## Exercice 2.5 — Authentification

Protège la view `create_product` de l'exercice 2.1 avec `@login_required`. Crée un utilisateur de test et vérifie qu'un utilisateur non connecté est redirigé vers la page de login.
