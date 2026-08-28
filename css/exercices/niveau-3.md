# CSS — Exercices niveau 3 (avancé)

## Exercice 3.1 — Composant vraiment réutilisable avec Container Queries

Construis une carte `.carte` qui passe d'un layout vertical (image au-dessus) à un layout horizontal (image à gauche) en fonction de la largeur de **son conteneur**, pas du viewport. Place cette même carte dans une sidebar étroite et dans une grille large sur la même page, et vérifie qu'elle s'adapte indépendamment dans chaque contexte.

## Exercice 3.2 — Cascade Layers pour intégrer une librairie tierce

Simule l'intégration d'une mini librairie de boutons tiers (`.btn { padding: 4px; background: gray; }`) dans un projet qui a déjà ses propres styles de boutons plus spécifiques. En utilisant `@layer`, garantis que **tes** styles de boutons gagnent toujours, sans toucher au CSS de la librairie ni utiliser `!important`.

## Exercice 3.3 — `:has()` pour un état de formulaire

Un formulaire doit afficher son bouton de soumission désactivé visuellement (opacité réduite, curseur `not-allowed`) tant qu'au moins un champ `input:invalid` existe à l'intérieur, sans aucun JavaScript.

## Exercice 3.4 — Accessibilité et mouvement

Une page contient plusieurs animations décoratives (`@keyframes`) déclenchées automatiquement à l'entrée dans le viewport. Mets en place le CSS nécessaire pour que ces animations soient neutralisées pour les utilisateurs ayant activé "réduire les animations" au niveau système, sans dupliquer les règles d'animation.

## Exercice 3.5 — Diagnostic de performance

On te donne ce CSS animé au scroll d'une liste de 200 éléments :

```css
.item {
  transition: top 300ms, left 300ms, width 300ms;
}
.item.active {
  top: 10px;
  left: 20px;
  width: 220px;
}
```

Explique pourquoi cette animation est probablement saccadée sur mobile, identifie précisément les propriétés fautives, et réécris une version équivalente visuellement mais performante.
