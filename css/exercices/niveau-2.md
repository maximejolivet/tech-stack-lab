# CSS — Exercices niveau 2 (intermédiaire)

## Exercice 2.1 — Grid + Flexbox combinés

Construis une page avec :
- Un layout global en **Grid** : header (toute la largeur), sidebar (200px fixe) + contenu principal (le reste), footer (toute la largeur)
- À l'intérieur du header, une barre en **Flexbox** avec logo, menu, et bouton de connexion alignés

## Exercice 2.2 — Variables CSS & thème

Mets en place des variables CSS (`--color-bg`, `--color-text`, `--color-primary`) sur `:root`, puis crée une bascule de thème sombre via `@media (prefers-color-scheme: dark)` qui redéfinit ces variables. Utilise-les dans au moins 3 composants différents (bouton, carte, titre).

## Exercice 2.3 — BEM

Reprends le composant "carte produit" suivant et renomme toutes les classes selon la convention BEM (Block__Element--Modifier) :

```html
<div class="card featured">
  <img class="card-image" src="produit.jpg">
  <h3 class="card-title">Nom du produit</h3>
  <span class="card-price">29,99 €</span>
</div>
```

## Exercice 2.4 — Animation performante

Crée une carte qui, au survol (`:hover`), se soulève légèrement (translation vers le haut de 4px) et affiche une ombre plus prononcée, en 200ms. La solution ne doit utiliser **que** des propriétés qui n'entraînent ni reflow ni repaint coûteux. Justifie ton choix de propriétés.

## Exercice 2.5 — Stacking context

Un `.tooltip` avec `z-index: 999` reste caché derrière une `.modal` qui a `z-index: 10`. Liste au moins deux causes possibles liées aux stacking contexts, et propose une correction.
