# Solutions HTML — Niveau 1

## 1.1 — Squelette valide

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ma première page</title>
</head>
<body>
</body>
</html>
```

## 1.2 — Hiérarchie de titres

```html
<h1>Mon blog</h1>

<article>
  <h2>Premier article</h2>
  <p>...</p>
</article>

<article>
  <h2>Deuxième article</h2>
  <p>...</p>
</article>
```

`<article>` est adapté ici car chaque post de blog a du sens indépendamment du reste de la page.

## 1.3 — Liste et liens

```html
<nav>
  <ul>
    <li><a href="/">Accueil</a></li>
    <li><a href="/blog">Blog</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```

## 1.4 — Image accessible

```html
<img src="chat.jpg" alt="Chat Siamois adulte au pelage crème et aux yeux bleus" width="800" height="533" loading="lazy">
```

`width`/`height` réservent l'espace avant chargement → pas de layout shift. `alt` décrit le contenu réel de l'image (pas juste "chat" ou "image").

## 1.5 — Repérer l'erreur

Problèmes :
1. `<div class="titre">` devrait être un vrai titre : `<h1>Bienvenue</h1>` (ou niveau adapté au contexte).
2. `<div onclick="goToContact()">` n'est ni focusable, ni activable au clavier, ni annoncé comme interactif par un lecteur d'écran → doit être un `<a href="/contact">` (navigation) ou un `<button>` (action JS).
3. `<img src="logo.png">` sans `alt` → invalide en accessibilité, ajouter `alt="Logo MonSite"` (ou `alt=""` si purement décoratif et déjà nommé ailleurs).

```html
<h1>Bienvenue</h1>
<a href="/contact">Contactez-nous</a>
<img src="logo.png" alt="Logo MonSite">
```
