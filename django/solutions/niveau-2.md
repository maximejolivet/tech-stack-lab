# Solutions — Niveau 2

## 2.1 — ModelForm

```python
# catalog/forms.py
from django import forms
from .models import Product

class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = ["name", "price", "in_stock"]
```

```python
# catalog/views.py
from django.shortcuts import render, redirect
from .forms import ProductForm

def create_product(request):
    if request.method == "POST":
        form = ProductForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("product-list")
    else:
        form = ProductForm()
    return render(request, "catalog/create.html", {"form": form})
```

## 2.2 — QuerySets

```python
from django.http import JsonResponse
from .models import Product

def product_stats(request):
    in_stock = Product.objects.filter(in_stock=True).order_by("price")
    out_of_stock_count = Product.objects.filter(in_stock=False).count()
    top_5_expensive = Product.objects.order_by("-price")[:5]

    return JsonResponse({
        "in_stock": list(in_stock.values("name", "price")),
        "out_of_stock_count": out_of_stock_count,
        "top_5_expensive": list(top_5_expensive.values("name", "price")),
    })
```

## 2.3 — Class-Based Views

```python
from django.views.generic import ListView, DetailView
from .models import Product

class ProductListView(ListView):
    model = Product
    template_name = "catalog/list.html"
    context_object_name = "products"
    queryset = Product.objects.filter(in_stock=True)

class ProductDetailView(DetailView):
    model = Product
    template_name = "catalog/detail.html"
    context_object_name = "product"
```

```python
# catalog/urls.py
from django.urls import path
from .views import ProductListView, ProductDetailView

urlpatterns = [
    path("", ProductListView.as_view(), name="product-list"),
    path("<int:pk>/", ProductDetailView.as_view(), name="product-detail"),
]
```

## 2.4 — Middleware custom

```python
# catalog/middleware.py
class PoweredByMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        response["X-Powered-By"] = "Django-Lab"
        return response
```

```python
# myshop/settings.py
MIDDLEWARE = [
    # ...
    "catalog.middleware.PoweredByMiddleware",
]
```

## 2.5 — Authentification

```python
from django.contrib.auth.decorators import login_required

@login_required
def create_product(request):
    # ... contenu inchangé de 2.1
    pass
```

```bash
python manage.py shell
```
```python
from django.contrib.auth.models import User
User.objects.create_user("testuser", password="testpass123")
```

Un utilisateur non connecté qui accède à la route protégée est redirigé vers `settings.LOGIN_URL` (par défaut `/accounts/login/`) avec un paramètre `?next=` pointant vers la page initialement demandée.
