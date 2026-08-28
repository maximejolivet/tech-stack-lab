# Django

## 1. Introduction

Django est un framework web Python "batteries included" : ORM, système d'authentification, interface d'administration et gestion des formulaires sont fournis nativement, sans devoir assembler des librairies tierces. Ce dossier suppose [`../python/`](../python/) déjà maîtrisé — c'est l'équivalent côté Python de [`../symfony/`](../symfony/) ou [`../laravel/`](../laravel/) côté PHP.

**À quoi sert-il ?**
- Construire des applications web complètes (sites, back-offices, APIs) rapidement grâce à ses composants intégrés.
- Servir de socle à des projets orientés données, grâce à son ORM mature et son admin auto-généré.
- Exposer des APIs REST via Django REST Framework (extension officieuse de facto, non intégrée au cœur mais quasi-standard).

**Où se situe-t-il dans une architecture web ?**
Django suit le pattern **MVT** (Model-View-Template) — une variation du MVC classique où Django lui-même joue le rôle de "Contrôleur" (le routage et le dispatch), et où la **View** Django correspond au Contrôleur Symfony/Laravel (pas à la vue au sens template). Comme Symfony/Laravel, Django tourne derrière un serveur WSGI (Gunicorn) ou ASGI (Uvicorn, pour le support async), lui-même généralement derrière Nginx.

**Avantages**
- "Batteries included" : ORM, migrations, admin, auth, forms — tout est cohérent et maintenu par le même projet, pas d'assemblage de packages tiers hétérogènes à faire cohabiter.
- Admin auto-généré à partir des modèles : gain de temps considérable pour un back-office interne, sans équivalent aussi direct côté Symfony/Laravel.
- Documentation officielle exceptionnellement complète et maintenue, communauté large et stable.

**Limites**
- Structure plus monolithique et "convention over configuration" que Symfony — moins modulaire à la carte que le système de bundles Symfony, plus proche de la philosophie Laravel dans son intégration.
- Pas de support REST natif dans le cœur : Django REST Framework est un package séparé (bien qu'incontournable en pratique), contrairement à API Platform intégré à l'écosystème Symfony.
- ORM Django moins flexible sur des requêtes très complexes que Doctrine ou l'Eloquent avancé — nécessite parfois de tomber en SQL brut (`.raw()`) pour des cas avancés.

## 2. Prérequis

- Python solide ([`../python/`](../python/)), en particulier la POO, les décorateurs et les compréhensions.
- Un framework MVC déjà pratiqué (Symfony ou Laravel) facilite la transposition des concepts (routing, ORM, migrations, templates) vers leurs équivalents Django.
- Une base de données installée (SQLite suffit pour démarrer, PostgreSQL recommandé pour un projet réaliste — voir [`../postgresql/`](../postgresql/)).

## 3. Rappel des bases 🟢

### 01 - Installation et structure de projet

**Explication** — `django-admin startproject` génère la structure racine (configuration, WSGI/ASGI, `manage.py`) ; `python manage.py startapp` génère une **app** — un module métier autonome (équivalent conceptuel d'un bundle Symfony ou d'un module Laravel), plusieurs apps composent un projet Django.

```bash
pip install django
django-admin startproject myproject
cd myproject
python manage.py startapp blog
```

```text
myproject/
├── manage.py              # CLI du projet (équivalent de bin/console Symfony ou artisan Laravel)
├── myproject/
│   ├── settings.py          # configuration centrale (DB, apps installées, middleware...)
│   ├── urls.py                # routes racine
│   └── wsgi.py / asgi.py        # point d'entrée serveur
└── blog/                          # une "app" = module métier
    ├── models.py
    ├── views.py
    ├── urls.py
    └── admin.py
```

**Bonne pratique** : une app par domaine métier cohérent (`blog`, `users`, `payments`), déclarée dans `INSTALLED_APPS` de `settings.py` — comparable à la découpe en bundles/modules des autres frameworks.

### 02 - Le pattern MVT (Model-View-Template)

**Explication** — **Model** (ORM, identique au concept Symfony/Laravel), **View** (fonction ou classe qui reçoit une requête et retourne une réponse — c'est le rôle du **Contrôleur** ailleurs), **Template** (rendu HTML, équivalent Twig/Blade). Le nom "View" est donc une source de confusion fréquente pour qui vient de Symfony/Laravel.

```python
# blog/views.py
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Bonjour le monde")
```

**Erreur fréquente** : chercher un dossier "Controller" — en Django, la logique de contrôleur vit dans `views.py`, et ce que Symfony/Laravel appellent "vue" (le template HTML) est dans `templates/`.

### 03 - Routing (urls.py)

**Explication** — Chaque app déclare ses routes dans son propre `urls.py`, inclus depuis le `urls.py` racine du projet via `include()` — approche modulaire proche des imports de routes par bundle en Symfony.

```python
# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("articles/<int:id>/", views.article_detail, name="article-detail"),
]

# myproject/urls.py
from django.urls import path, include

urlpatterns = [
    path("blog/", include("blog.urls")),
]
```

**Bonne pratique** : toujours nommer ses routes (`name="..."`) et les référencer via `reverse()`/`{% url %}` plutôt qu'en dur — comme les noms de route Symfony, ça évite de casser des liens lors d'un changement d'URL.

### 04 - Views (function-based)

**Explication** — Une view est une fonction Python qui prend au minimum `request` en paramètre et retourne un objet `HttpResponse` (ou une sous-classe : `JsonResponse`, `render()` qui combine template + contexte).

```python
from django.shortcuts import render, get_object_or_404
from .models import Article

def article_detail(request, id):
    article = get_object_or_404(Article, pk=id)
    return render(request, "blog/detail.html", {"article": article})
```

**Cas d'usage** : `get_object_or_404` est l'idiome standard pour lever automatiquement une 404 propre si l'objet n'existe pas — évite un `try/except DoesNotExist` répété partout.

### 05 - Templates (Django Template Language)

**Explication** — Syntaxe `{{ variable }}` et `{% tag %}`, volontairement moins permissive que du Python pur (pas d'exécution de code arbitraire dans un template) — même philosophie que Twig, syntaxe très proche.

```html
<!-- blog/templates/blog/detail.html -->
<h1>{{ article.title }}</h1>
<p>{{ article.content|truncatewords:30 }}</p>

{% if article.published %}
    <span>Publié le {{ article.published_at|date:"d/m/Y" }}</span>
{% endif %}

{% for comment in article.comments.all %}
    <p>{{ comment.text }}</p>
{% endfor %}
```

**Bonne pratique** : logique métier dans la view ou le model, jamais dans le template — le DTL décourage volontairement la logique complexe (pas d'affectation de variable arbitraire, filtres limités).

### 06 - Models et ORM

**Explication** — Une classe héritant de `models.Model` décrit une table. Chaque attribut de classe typé (`CharField`, `IntegerField`, `ForeignKey`...) devient une colonne — équivalent direct d'une entité Doctrine ou d'un modèle Eloquent.

```python
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    published = models.BooleanField(default=False)
    published_at = models.DateTimeField(null=True, blank=True)
    author = models.ForeignKey("auth.User", on_delete=models.CASCADE)

    def __str__(self):
        return self.title
```

**Erreur fréquente** : oublier `on_delete` sur un `ForeignKey` — obligatoire en Django (contrairement à certains ORM où une valeur par défaut existe), il faut choisir explicitement le comportement (`CASCADE`, `SET_NULL`, `PROTECT`...).

### 07 - Migrations

**Explication** — Django génère les migrations à partir des changements détectés dans `models.py` (`makemigrations`), puis les applique à la base (`migrate`) — workflow proche de Doctrine Migrations ou Laravel Migrations, mais la génération automatique par diff est plus poussée nativement.

```bash
python manage.py makemigrations blog   # génère le fichier de migration à partir du diff des models
python manage.py migrate                 # applique les migrations en attente
python manage.py sqlmigrate blog 0001      # affiche le SQL généré, sans l'exécuter
```

**Bonne pratique** : toujours relire une migration auto-générée avant de la commiter, en particulier sur une modification de colonne existante (changement de type, ajout d'une contrainte NOT NULL sans valeur par défaut sur une table déjà peuplée).

### 08 - Interface d'administration

**Explication** — En enregistrant un modèle dans `admin.py`, Django génère automatiquement une interface CRUD complète (liste, création, édition, suppression, recherche, filtres) — sans équivalent aussi immédiat côté Symfony (EasyAdmin s'en approche, en package séparé) ou Laravel (Nova, payant).

```python
# blog/admin.py
from django.contrib import admin
from .models import Article

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ["title", "published", "published_at"]
    list_filter = ["published"]
    search_fields = ["title"]
```

**Cas d'usage** : idéal pour un back-office interne rapide (gestion de contenu, modération) sans développer d'interface dédiée — à ne pas exposer tel quel comme back-office client final sans personnalisation.

## 4. Concepts intermédiaires 🟡

- **QuerySets et l'ORM en profondeur** : un `QuerySet` est **paresseux** (lazy) — la requête SQL n'est exécutée qu'au moment où le résultat est réellement consommé (itération, `list()`, `len()`), ce qui permet de chaîner des filtres sans coût intermédiaire.

```python
articles = Article.objects.filter(published=True).exclude(author__isnull=True).order_by("-published_at")
# aucune requête SQL n'est encore partie ici

for article in articles:  # la requête SQL part seulement maintenant
    print(article.title)
```

- **Forms et ModelForms** : `Form` valide des données arbitraires, `ModelForm` génère automatiquement les champs et la validation à partir d'un modèle — équivalent des FormType Symfony, avec un couplage direct au modèle par défaut.

```python
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ["title", "content", "published"]

def create_article(request):
    if request.method == "POST":
        form = ArticleForm(request.POST)
        if form.is_valid():
            form.save()
    else:
        form = ArticleForm()
    return render(request, "blog/create.html", {"form": form})
```

- **Class-Based Views (CBV)** : alternative aux function-based views pour les cas CRUD standards, via des classes génériques prêtes à l'emploi (`ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView`) — réduit le boilerplate pour les cas courants, au prix d'une indirection à apprendre.

```python
from django.views.generic import ListView, DetailView

class ArticleListView(ListView):
    model = Article
    template_name = "blog/list.html"
    context_object_name = "articles"
    queryset = Article.objects.filter(published=True)

class ArticleDetailView(DetailView):
    model = Article
    template_name = "blog/detail.html"
```

- **Middleware** : composants exécutés sur chaque requête/réponse, avant/après la view — équivalent direct des middlewares Symfony/Laravel/Express.

```python
class RequestTimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        import time
        start = time.perf_counter()
        response = self.get_response(request)
        elapsed = time.perf_counter() - start
        response["X-Response-Time"] = f"{elapsed:.4f}s"
        return response
```

- **Authentification** : `django.contrib.auth` fournit un modèle `User`, un système de sessions, `login()`/`logout()`, des décorateurs (`@login_required`) et un système de permissions/groupes intégré — équivalent du Security bundle Symfony ou de l'auth scaffolding Laravel, activé par défaut dès la création du projet.

```python
from django.contrib.auth.decorators import login_required

@login_required
def dashboard(request):
    return render(request, "dashboard.html")
```

- **Django REST Framework (DRF)** : package séparé mais quasi-standard pour exposer des APIs REST — `Serializer` (transforme modèle ↔ JSON, validation incluse), `ViewSet` (CRUD API générique) — comparable à API Platform côté Symfony.

```python
from rest_framework import serializers, viewsets

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ["id", "title", "content", "published"]

class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
```

## 5. Concepts avancés 🟠🔴

- **Signals** : mécanisme de découplage permettant à du code de réagir à un événement du framework (`post_save`, `pre_delete`...) sans coupler directement l'émetteur et le récepteur — proche du principe des Events Symfony ou des Events/Listeners Laravel.

```python
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=Article)
def notify_on_publish(sender, instance, created, **kwargs):
    if instance.published:
        print(f"Article publié : {instance.title}")
```

- **Managers custom** : personnaliser `objects` (le manager par défaut d'un modèle) pour encapsuler des requêtes fréquentes réutilisables — évite de dupliquer des `.filter(...)` complexes dans tout le codebase.

```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(published=True)

class Article(models.Model):
    # ...
    objects = models.Manager()          # manager par défaut
    published_objects = PublishedManager()  # manager custom

Article.published_objects.all()  # équivalent de Article.objects.filter(published=True)
```

- **Optimisation des requêtes N+1** : `select_related` (jointure SQL pour une relation `ForeignKey`/`OneToOne`, une seule requête) et `prefetch_related` (requête séparée puis jointure en Python, pour les relations `ManyToMany`/inverse) — équivalent du problème et des solutions rencontrés avec Doctrine/Eloquent.

```python
# N+1 : une requête pour les articles, puis une par article pour son auteur
articles = Article.objects.all()
for a in articles:
    print(a.author.username)  # requête SQL supplémentaire à chaque itération

# Corrigé : une seule requête avec jointure
articles = Article.objects.select_related("author").all()
```

- **Caching** : le framework de cache intégré (`django.core.cache`) supporte plusieurs backends (mémoire locale, Memcached, Redis — voir [`../redis/`](../redis/)), avec du cache de vue entière, de fragment de template, ou de valeur arbitraire — équivalent conceptuel du composant Cache Symfony.
- **Tâches asynchrones avec Celery** : Django lui-même reste synchrone par requête (hors vues `async def` récentes) ; pour du traitement différé/en arrière-plan (envoi d'email, traitement lourd), Celery est le standard de facto — comparable à Symfony Messenger ou aux Jobs/Queues Laravel.
- **Déploiement WSGI vs ASGI** : WSGI (Gunicorn) est le modèle synchrone classique (une requête = un thread/process bloquant) ; ASGI (Uvicorn/Daphne) permet des vues asynchrones natives (`async def` dans `views.py`) pour du code I/O-bound à forte concurrence, en s'appuyant sur `asyncio` ([`../python/`](../python/), section avancée).

## 6. Commandes / syntaxe à connaître

```bash
django-admin startproject myproject       # créer un nouveau projet
python manage.py startapp blog              # créer une nouvelle app
python manage.py runserver                     # serveur de dev
python manage.py makemigrations                # générer les migrations depuis les models
python manage.py migrate                        # appliquer les migrations
python manage.py createsuperuser                  # créer un compte admin
python manage.py shell                              # REPL avec le contexte Django chargé
python manage.py test                                 # lancer les tests
```

```python
# Syntaxe essentielle à avoir sous les doigts
Article.objects.filter(published=True).order_by("-published_at")
Article.objects.select_related("author").prefetch_related("comments")
get_object_or_404(Article, pk=id)
render(request, "template.html", {"key": value})
path("route/<int:id>/", views.detail, name="detail")
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Blog minimal avec back-office admin**

Construire une petite application Django qui doit :
- Modéliser `Article` (titre, contenu, statut publié, date, auteur `ForeignKey` vers `User`) et `Comment` (`ForeignKey` vers `Article`).
- Exposer une `ListView` (articles publiés uniquement) et une `DetailView` (avec ses commentaires, en utilisant `prefetch_related` pour éviter le N+1).
- Fournir un formulaire `ModelForm` pour ajouter un commentaire, protégé par `@login_required`.
- Enregistrer `Article` et `Comment` dans `admin.py` avec `list_display` et `list_filter` pertinents.
- Ajouter un signal `post_save` qui affiche un message en console quand un article passe à `published=True`.

Objectif : mobiliser models/ORM, views (function ou class-based), forms, admin et signals dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux (structure de projet, pattern MVT, routing)
- [ ] Savoir créer un projet et une app Django
- [ ] Maîtriser la syntaxe principale (models, views, templates, migrations)
- [ ] Comprendre les concepts importants (QuerySets, forms, CBV, middleware, auth)
- [ ] Savoir debugger (Django Debug Toolbar, `python manage.py shell`, logs)
- [ ] Connaître les bonnes pratiques (select_related/prefetch_related, migrations relues avant commit)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (signals, managers custom, cache, Celery, ASGI)

## 10. Ressources

- [Documentation officielle Django](https://docs.djangoproject.com/) — référence complète, l'une des meilleures documentations de framework toutes technologies confondues.
- [Django REST Framework — documentation](https://www.django-rest-framework.org/) pour les APIs.
- [Classy Class-Based Views](https://ccbv.co.uk/) — référence pratique des CBV et de leurs attributs/méthodes.
- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/) — indispensable pour visualiser les requêtes SQL générées et repérer les N+1.
- [roadmap.sh — Django](https://roadmap.sh/django) — vue d'ensemble du parcours d'apprentissage.
