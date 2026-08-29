# Exercices Accessibility & Performance — Niveau 1 (Bases)

## Exercice 1 — Corriger un div cliquable

Voici un bouton mal implémenté :

```html
<div class="btn" onclick="addToCart()">Ajouter au panier</div>
```

Explique les deux problèmes d'accessibilité, puis corrige le code.

## Exercice 2 — Texte alternatif

Voici trois images. Pour chacune, écris l'attribut `alt` approprié (ou justifie un `alt` vide) : (a) le logo de l'entreprise dans le header, qui fait aussi office de lien vers l'accueil ; (b) une icône décorative de séparation entre deux sections ; (c) une photo de produit dans une fiche produit.

## Exercice 3 — Vérifier un contraste

Un texte gris `#999999` est affiché sur un fond blanc `#FFFFFF`, en `14px` normal. Ce texte respecte-t-il le contraste minimal WCAG AA (4.5:1) ? Explique comment tu vérifierais cela concrètement.

## Exercice 4 — ARIA mal utilisé

```html
<div role="button" onclick="submit()">Valider</div>
```

Liste ce qui manque pour que ce composant soit réellement utilisable au clavier, en plus du `role`.

## Exercice 5 — Réserver l'espace d'une image

Voici une balise image sans dimensions :

```html
<img src="banner.jpg" alt="Bannière promotionnelle">
```

Explique le problème de performance perçue que cela cause, puis corrige le code.
