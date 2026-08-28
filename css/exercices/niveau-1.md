# CSS — Exercices niveau 1 (bases)

## Exercice 1.1 — Sélecteurs et spécificité

Étant donné ce HTML :

```html
<p class="texte">Un paragraphe</p>
<p id="intro" class="texte">Un autre paragraphe</p>
```

Écris trois règles CSS différentes qui ciblent respectivement :
1. Tous les `<p>`
2. Uniquement les éléments avec la classe `texte`
3. Uniquement l'élément avec l'id `intro`

Puis réponds : si les trois règles définissent une `color` différente sur le paragraphe `#intro`, laquelle s'applique et pourquoi ?

## Exercice 1.2 — Box model

Crée un `.encart` de largeur totale 300px (bordures incluses), avec 20px de padding et une bordure de 2px, **sans** que le contenu ne dépasse les 300px prévus. Explique la propriété clé utilisée.

## Exercice 1.3 — Flexbox simple

Construis une barre de navigation horizontale avec un logo à gauche et 3 liens à droite, alignés verticalement au centre, en utilisant uniquement Flexbox (pas de `float`, pas de `position`).

## Exercice 1.4 — Responsive de base

Un titre doit faire `1.5rem` sur mobile et `2.5rem` à partir de 768px de large. Écris le CSS en mobile-first.

## Exercice 1.5 — Display

Explique (en commentaire CSS) pourquoi ce code ne produit aucun effet visible, et corrige-le :

```css
.mon-lien {
  display: inline;
  width: 200px;
  height: 50px;
}
```
