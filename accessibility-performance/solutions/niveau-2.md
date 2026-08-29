# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```html
<picture>
  <source srcset="produit-400.avif 400w, produit-800.avif 800w, produit-1200.avif 1200w"
          type="image/avif" sizes="(max-width: 600px) 400px, 800px">
  <source srcset="produit-400.webp 400w, produit-800.webp 800w, produit-1200.webp 1200w"
          type="image/webp" sizes="(max-width: 600px) 400px, 800px">
  <img src="produit-800.jpg" alt="Casque audio sans fil noir, vue de face">
</picture>
```

## Exercice 2

```html
<div role="status" aria-live="polite">
  Votre message a bien été envoyé.
</div>
```

`aria-live="polite"` fait annoncer automatiquement par le lecteur d'écran tout changement de contenu dans ce conteneur, sans interrompre ce que l'utilisateur était en train d'écouter.

## Exercice 3

Sans `font-display`, le navigateur peut masquer le texte pendant le chargement de la police custom (Flash of Invisible Text) — l'utilisateur voit une page vide de texte pendant ce laps de temps.

```css
@font-face {
  font-family: 'Custom';
  src: url('font.woff2') format('woff2');
  font-display: swap;
}
```

`font-display: swap` affiche immédiatement le texte avec une police de secours, puis bascule vers la police custom une fois chargée.

## Exercice 4

Le niveau A couvre les critères minimaux indispensables (ex. `alt` sur les images), AA ajoute des exigences plus larges couvrant la majorité des situations de handicap (contraste 4.5:1, navigation clavier complète) et AAA couvre les exigences les plus strictes, difficiles à satisfaire intégralement sans compromis significatif de design (ex. contraste 7:1). Le niveau AA est le standard professionnel généralement visé, y compris dans la plupart des obligations légales (RGAA, ADA).

## Exercice 5

Ajouter `loading="lazy"` sur ces images retarde leur téléchargement jusqu'à ce qu'elles approchent du viewport visible, réduisant le nombre de requêtes et le poids total téléchargé au chargement initial de la page.

```html
<img src="produit-recommande.jpg" alt="Nom du produit" loading="lazy" width="300" height="300">
```
