# Solutions — Niveau 1 (Bases)

## Exercice 1

Problèmes : (1) un `<div>` n'est pas nativement focusable au clavier — un utilisateur au clavier ne peut pas l'atteindre avec `Tab` ; (2) un lecteur d'écran ne l'annonce pas comme un bouton actionnable, seulement comme du texte.

```html
<button class="btn" onclick="addToCart()">Ajouter au panier</button>
```

## Exercice 2

(a) Logo/lien vers l'accueil : `alt="Accueil - Nom de l'entreprise"` (décrit la destination du lien, pas juste "logo").
(b) Icône décorative : `alt=""` (vide mais présent, pour être explicitement ignorée par le lecteur d'écran).
(c) Photo de produit : `alt="Casque audio sans fil noir, vue de face"` (description concrète et utile du contenu visuel).

## Exercice 3

Non, `#999999` sur fond blanc `#FFFFFF` donne un ratio de contraste d'environ 2.85:1, en dessous du minimum WCAG AA de 4.5:1 pour du texte standard. Pour vérifier concrètement : utiliser l'inspecteur de couleur de Chrome DevTools (qui affiche le ratio directement au survol d'un texte) ou un outil dédié comme WebAIM Contrast Checker en saisissant les deux couleurs.

## Exercice 4

Il manque : `tabindex="0"` pour que l'élément soit atteignable via `Tab` (un `<div>` n'est pas focusable par défaut), et un gestionnaire d'événement clavier qui déclenche `submit()` sur `Enter` et `Espace` — ARIA ne fait que déclarer le rôle sémantique, il n'ajoute aucun comportement.

## Exercice 5

Sans dimensions déclarées, le navigateur ne connaît l'espace que l'image occupera qu'une fois celle-ci téléchargée : le contenu situé en dessous "saute" visuellement quand l'image apparaît (Cumulative Layout Shift), pouvant faire cliquer l'utilisateur au mauvais endroit.

```html
<img src="banner.jpg" alt="Bannière promotionnelle" width="1200" height="400">
```
