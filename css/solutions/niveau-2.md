# CSS — Solutions niveau 2

## 2.1 — Grid + Flexbox combinés

```css
.page {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  min-height: 100vh;
}
.page__header { grid-area: header; }
.page__sidebar { grid-area: sidebar; }
.page__main { grid-area: main; }
.page__footer { grid-area: footer; }

.page__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

Grid structure le squelette de page (deux axes : lignes ET colonnes) ; Flexbox aligne les éléments à l'intérieur du header (un seul axe).

## 2.2 — Variables CSS & thème

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-primary: #2563eb;
}
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #111827;
    --color-text: #f3f4f6;
    --color-primary: #60a5fa;
  }
}

body { background: var(--color-bg); color: var(--color-text); }
.btn { background: var(--color-primary); color: white; }
.carte { background: var(--color-bg); border: 1px solid var(--color-primary); }
h1 { color: var(--color-primary); }
```

## 2.3 — BEM

```html
<div class="carte-produit carte-produit--featured">
  <img class="carte-produit__image" src="produit.jpg">
  <h3 class="carte-produit__titre">Nom du produit</h3>
  <span class="carte-produit__prix">29,99 €</span>
</div>
```

`carte-produit` = Block, `__image`/`__titre`/`__prix` = Elements, `--featured` = Modifier. Chaque classe reste à spécificité (0,1,0), aucun nesting de sélecteur nécessaire.

## 2.4 — Animation performante

```css
.carte {
  transition: transform 200ms ease-out, box-shadow 200ms ease-out;
}
.carte:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgb(0 0 0 / 0.15);
}
```

`transform` et `box-shadow` (via un changement de valeur, pas de propriété géométrique) sont gérés par le compositeur/GPU sans recalcul de layout — contrairement à `top`/`margin-top` qui déclencheraient un reflow à chaque frame.

## 2.5 — Stacking context

Causes possibles :
1. Un ancêtre du `.tooltip` a une `opacity < 1`, un `transform`, ou un `position` avec `z-index` — il crée un nouveau stacking context qui plafonne tous ses descendants, quel que soit leur `z-index`.
2. `.tooltip` est positionné en `static` (donc `z-index` n'a aucun effet — `z-index` ne s'applique qu'aux éléments positionnés).

Correction : donner à `.tooltip` un `position: fixed` ou `absolute` explicite, et/ou le sortir du conteneur parent qui crée le stacking context limitant (ex. via un portail dans le DOM, ou en supprimant le `transform`/`opacity` inutile sur l'ancêtre).
