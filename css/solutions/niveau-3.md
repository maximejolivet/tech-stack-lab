# CSS — Solutions niveau 3

## 3.1 — Container Queries

```css
.carte-wrapper {
  container-type: inline-size;
  container-name: carte;
}
.carte {
  display: flex;
  flex-direction: column;
}
.carte__image { width: 100%; }

@container carte (min-width: 400px) {
  .carte {
    flex-direction: row;
  }
  .carte__image {
    width: 40%;
  }
}
```

Chaque `.carte-wrapper` définit son propre contexte de conteneur : la même carte passe en horizontal dans une grille large (colonne > 400px) et reste verticale dans une sidebar étroite, sans media query ni JS — la logique est portée par le composant lui-même, pas par le viewport global.

## 3.2 — Cascade Layers

```css
@layer libs, app;

/* CSS de la librairie tierce, non modifiable, placé dans la couche "libs" */
@layer libs {
  .btn { padding: 4px; background: gray; }
}

/* Nos styles, placés APRÈS dans l'ordre des couches déclarées */
@layer app {
  .btn { padding: 12px 20px; background: var(--color-primary); }
}
```

L'ordre de déclaration des `@layer` (première ligne) fixe la priorité : `app` gagne toujours sur `libs`, même si la règle de la librairie était plus spécifique ou déclarée plus bas dans le fichier final — sans `!important`, sans toucher au code tiers.

## 3.3 — `:has()` sans JS

```css
form:has(input:invalid) .btn-submit {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}
```

`form:has(input:invalid)` sélectionne le `<form>` dès qu'un de ses descendants est invalide (validation HTML native via `required`, `type="email"`, `pattern`, etc.), et on stylise le bouton en conséquence — purement déclaratif.

## 3.4 — Accessibilité et mouvement

```css
.animation-entree {
  animation: fade-in 400ms ease-out;
}

@media (prefers-reduced-motion: reduce) {
  .animation-entree {
    animation: none;
  }
}
```

Approche généralisable sans dupliquer chaque `@keyframes` :

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Cette règle globale neutralise toutes les animations/transitions du site en une seule déclaration, sans devoir retoucher chaque composant individuellement.

## 3.5 — Diagnostic de performance

Propriétés fautives : `top`, `left`, `width` sont des propriétés **géométriques** — chaque changement déclenche un **reflow** (recalcul du layout de la page, potentiellement de tous les éléments après celui modifié) suivi d'un **repaint**. Avec 200 éléments animés simultanément sur 300ms, le navigateur recalcule le layout à quasi chaque frame → jank visible, particulièrement sur mobile (CPU plus faible).

Version performante, équivalente visuellement, en `transform` :

```css
.item {
  transition: transform 300ms;
}
.item.active {
  transform: translate(20px, 10px) scaleX(1.375); /* 220px / largeur initiale, ex. 160px */
}
```

`transform` (translation + scale) est géré par le compositeur graphique sur son propre thread, sans recalcul de layout ni repaint du reste de la page — l'animation reste fluide même avec 200 éléments.
