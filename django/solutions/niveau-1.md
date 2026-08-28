# Solutions — Niveau 1

## 1.1 — Premier projet

```bash
django-admin startproject myshop
cd myshop
python manage.py startapp catalog
```

```python
# myshop/settings.py
INSTALLED_APPS = [
    # ...
    "catalog",
]
```

```bash
python manage.py runserver
```

## 1.2 — View minimale

```python
# catalog/views.py
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Bienvenue dans la boutique")
```

```python
# catalog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.hello, name="hello"),
]
```

```python
# myshop/urls.py
from django.urls import path, include

urlpatterns = [
    path("", include("catalog.urls")),
]
```

## 1.3 — Model et migration

```python
# catalog/models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    in_stock = models.BooleanField(default=True)
```

```bash
python manage.py makemigrations catalog
python manage.py migrate
python manage.py sqlmigrate catalog 0001
```

## 1.4 — Template avec boucle

```python
# catalog/views.py
from django.shortcuts import render
from .models import Product

def product_list(request):
    products = Product.objects.all()
    return render(request, "catalog/list.html", {"products": products})
```

```html
<!-- catalog/templates/catalog/list.html -->
<ul>
{% for product in products %}
    <li>{{ product.name }} — {{ product.price }} €</li>
{% endfor %}
</ul>
```

## 1.5 — Admin

```python
# catalog/admin.py
from django.contrib import admin
from .models import Product

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ["name", "price", "in_stock"]
```

```bash
python manage.py createsuperuser
python manage.py runserver
# puis se rendre sur /admin/
```
