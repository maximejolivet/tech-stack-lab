# Solutions — Niveau 3

## 3.1 — N+1 et select_related/prefetch_related

```python
# catalog/models.py
class Category(models.Model):
    name = models.CharField(max_length=100)

class Product(models.Model):
    # ... champs existants
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
```

```python
# Sans optimisation : 1 requête pour les produits + 1 requête par produit pour sa catégorie (N+1)
def product_list_naive(request):
    products = Product.objects.all()
    for p in products:
        print(p.category.name)  # requête SQL supplémentaire à chaque itération
    return render(request, "catalog/list.html", {"products": products})

# Corrigé : une seule requête avec jointure SQL
def product_list_optimized(request):
    products = Product.objects.select_related("category").all()
    for p in products:
        print(p.category.name)  # aucune requête supplémentaire, déjà chargé par la jointure
    return render(request, "catalog/list.html", {"products": products})
```

Avec 20 produits, la version naïve exécute 21 requêtes (1 + 20), la version optimisée en exécute 1 seule — vérifiable via `len(django.db.connection.queries)` avec `DEBUG=True`, ou visuellement dans Django Debug Toolbar.

## 3.2 — Signal

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from .models import Product

@receiver(pre_save, sender=Product)
def notify_out_of_stock(sender, instance, **kwargs):
    if instance.pk:  # produit existant, pas une création
        previous = Product.objects.filter(pk=instance.pk).first()
        if previous and previous.in_stock and not instance.in_stock:
            print(f"Produit passé en rupture de stock : {instance.name}")
```

Comme les Events Symfony (ou les Events/Listeners Laravel), les signals découplent l'émetteur (le code qui sauvegarde un `Product`) du récepteur (la notification) : le code métier qui modifie un produit n'a aucune connaissance de la logique de notification, qui peut être ajoutée, retirée ou modifiée sans toucher au code qui déclenche l'événement.

## 3.3 — Manager custom

```python
from django.db import models

class InStockManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(in_stock=True)

class Product(models.Model):
    # ... champs existants
    objects = models.Manager()            # manager par défaut, inchangé
    in_stock_objects = InStockManager()      # manager custom

# Usage :
Product.in_stock_objects.all()  # équivalent de Product.objects.filter(in_stock=True)
```

## 3.4 — API REST avec DRF

```bash
pip install djangorestframework
```

```python
# myshop/settings.py
INSTALLED_APPS = [
    # ...
    "rest_framework",
]
```

```python
# catalog/serializers.py
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "price", "in_stock"]
```

```python
# catalog/views.py
from rest_framework import viewsets
from .serializers import ProductSerializer
from .models import Product

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

```python
# myshop/urls.py
from rest_framework.routers import DefaultRouter
from catalog.views import ProductViewSet

router = DefaultRouter()
router.register("products", ProductViewSet)

urlpatterns = [
    # ...
    path("api/", include(router.urls)),
]
```

```bash
curl http://localhost:8000/api/products/
curl -X POST http://localhost:8000/api/products/ -d "name=Test&price=9.99&in_stock=true"
```
