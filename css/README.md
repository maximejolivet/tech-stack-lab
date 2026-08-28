# CSS

## 1. Introduction

CSS (Cascading Style Sheets) sépare le **contenu** (HTML) de la **présentation** (mise en forme, layout, comportement visuel). Le navigateur construit un DOM à partir du HTML, puis un **CSSOM** à partir des feuilles de style, et les combine en un *render tree* qui détermine ce qui est réellement peint à l'écran.

**Où se situe CSS dans l'architecture web** : couche présentation, exécutée côté client par le moteur de rendu du navigateur (Blink, WebKit, Gecko). Le JS peut manipuler le CSS dynamiquement (`classList`, `style`, variables CSS) mais CSS reste déclaratif : on décrit un état, pas une suite d'instructions.

**Avantages** : déclaratif et prévisible une fois la cascade comprise, performant (le navigateur optimise le rendu), cache-friendly, aucune dépendance à un framework.
**Limites** : la cascade peut devenir difficile à maîtriser à grande échelle sans convention (d'où BEM, CSS Modules...), pas de vraie logique de programmation (variables limitées, pas de boucles natives — Sass comble ça côté préprocesseur).

> Ce dossier couvre le **CSS natif moderne**. L'approche *utility-first* (Tailwind CSS) est traitée séparément dans `tailwindcss/` — comprendre le CSS natif en profondeur est un prérequis pour l'utiliser efficacement.

## 2. Prérequis

- Bases HTML (structure d'un document, éléments block/inline) — voir `html/`
- Aucune connaissance CSS préalable requise, ce dossier repart des fondamentaux

## 3. Rappel des bases 🟢

### 01 - Syntaxe & sélecteurs

```css
selecteur {
  propriete: valeur;
}

/* type, classe, id, attribut, universel */
p { }
.carte { }
#header { }
[data-state="open"] { }
* { }

/* combinateurs */
.parent > .enfant     /* enfant direct */
.a .b                 /* descendant */
.a + .b               /* frère adjacent immédiat */
.a ~ .b               /* frères suivants */
```

**Cas d'usage** : `.classe` pour du réutilisable, `#id` pour un élément unique (souvent utilisé comme ancre JS/anchor plutôt que style), attribut pour cibler un état sans classe dédiée.
**Erreur fréquente** : abuser des sélecteurs d'éléments génériques (`div span p`) → couplage fort à la structure HTML, casse au moindre refactor.
**Bonne pratique** : cibler par classe plutôt que par balise ou id pour le style ; garder les sélecteurs courts (2-3 niveaux max).

### 02 - Cascade, spécificité, héritage

C'est **le** concept fondamental de CSS : quand plusieurs règles s'appliquent au même élément, laquelle gagne ?

1. **Origine et importance** : `!important` d'un style utilisateur/auteur > styles auteur normaux > styles navigateur par défaut.
2. **Spécificité** (poids d'un sélecteur), calculée en (id, classe/attribut/pseudo-classe, élément/pseudo-élément) :

```css
#id                    /* (1,0,0) */
.classe.autre-classe   /* (0,2,0) */
div p                  /* (0,0,2) */
div.classe             /* (0,1,1) */
```

3. **Ordre d'apparition** : à spécificité égale, la dernière règle déclarée gagne.

**Héritage** : certaines propriétés (typographie : `color`, `font-*`, `line-height`) se transmettent aux enfants par défaut ; d'autres non (`margin`, `border`, `background`). `inherit`, `initial`, `unset`, `revert` permettent de forcer un comportement.

**Erreur fréquente** : empiler des `!important` pour "gagner" la cascade → dette technique immédiate, plus aucune règle n'est prévisible.
**Bonne pratique** : garder une spécificité basse et homogène (classes uniquement), n'utiliser `!important` que pour des utilitaires volontairement prioritaires (ex. `.hidden { display: none !important; }`).

### 03 - Box model

Chaque élément est une boîte : `content` → `padding` → `border` → `margin`.

```css
.box {
  box-sizing: border-box; /* width/height incluent padding + border */
  width: 200px;
  padding: 16px;
  border: 1px solid #ccc;
  margin: 8px;
}
```

Par défaut (`box-sizing: content-box`), `width` ne compte que le contenu : ajouter un padding **agrandit** la boîte au-delà de `width`. C'est la source n°1 de layouts cassés chez les débutants.

**Bonne pratique** : reset universel en tête de feuille de style :

```css
*, *::before, *::after { box-sizing: border-box; }
```

### 04 - Unités

| Unité | Relative à | Cas d'usage |
|---|---|---|
| `px` | fixe | bordures fines, valeurs qui ne doivent jamais scaler |
| `%` | parent | largeurs fluides |
| `em` | `font-size` de l'élément (ou parent pour d'autres props) | espacements proportionnels au texte local |
| `rem` | `font-size` de `:root` (html) | typographie et espacements cohérents dans toute l'app |
| `vw` / `vh` | viewport | sections plein écran, hero |
| `fr` | espace disponible en Grid | répartition de colonnes/lignes |

**Erreur fréquente** : chaîner des `em` sur des éléments imbriqués → effet cumulatif imprévisible (1.2em dans un 1.2em dans un 1.2em...).
**Bonne pratique** : `rem` par défaut pour tout ce qui touche à la taille/l'espacement, `em` seulement pour du proportionnel local volontaire (ex. taille d'icône liée au `font-size` du bouton qui la contient).

### 05 - Couleurs & typographie

```css
color: #1a1a1a;
color: rgb(26 26 26 / 0.8);   /* syntaxe moderne avec alpha */
color: hsl(220 90% 56%);

font-family: system-ui, -apple-system, sans-serif; /* toujours une pile de fallback */
font-size: 1rem;
font-weight: 600;
line-height: 1.5;             /* sans unité = proportionnel au font-size */
```

**Bonne pratique** : `line-height` sans unité (ex. `1.5`, pas `1.5em`) pour qu'il reste proportionnel si `font-size` change via media query.

### 06 - Display

```css
display: block;         /* prend toute la largeur, saut de ligne (div, p, section) */
display: inline;        /* dans le flux du texte, pas de width/height (span, a) */
display: inline-block;  /* dans le flux mais accepte width/height/margin vertical */
display: none;          /* retiré du rendu ET du flux */
```

**Erreur fréquente** : appliquer `width`/`height` sur un élément `inline` (ex. `span`) et s'étonner que ça n'ait aucun effet.
**Bonne pratique** : `visibility: hidden` (garde l'espace occupé) vs `display: none` (retire l'espace) — choisir selon le besoin de layout stable.

### 07 - Position

```css
position: static;    /* comportement par défaut, dans le flux normal */
position: relative;  /* décalé par rapport à sa position normale, garde sa place dans le flux */
position: absolute;   /* sorti du flux, positionné par rapport à l'ancêtre positionné le plus proche */
position: fixed;      /* positionné par rapport au viewport, ignore le scroll */
position: sticky;     /* hybride : relative jusqu'à un seuil de scroll, puis fixed */
```

**Erreur fréquente** : un `position: absolute` qui "s'échappe" de sa carte parce que le parent n'a pas de `position: relative`.
**Bonne pratique** : toujours poser `position: relative` sur le conteneur avant d'y positionner un enfant en `absolute`.

### 08 - Flexbox (axe unique)

```css
.container {
  display: flex;
  flex-direction: row;          /* ou column */
  justify-content: space-between; /* alignement sur l'axe principal */
  align-items: center;            /* alignement sur l'axe secondaire */
  gap: 16px;
  flex-wrap: wrap;
}
.item {
  flex: 1 1 200px; /* grow shrink basis */
}
```

**Cas d'usage** : barres de navigation, listes de cartes qui s'alignent sur une ligne, centrage vertical/horizontal.
**Erreur fréquente** : confondre `justify-content` (axe principal) et `align-items` (axe secondaire) quand `flex-direction` change.
**Bonne pratique** : `gap` plutôt que des `margin` sur les enfants pour l'espacement — évite les marges à gérer sur le premier/dernier élément.

### 09 - Grid (deux axes)

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 24px;
}
.item {
  grid-column: span 2;
}
```

**Cas d'usage** : mises en page globales (header/sidebar/main/footer), grilles de cartes avec alignement dans les deux dimensions.
**Flexbox vs Grid** : Flexbox = distribution sur un axe (contenu qui dicte la taille) ; Grid = structure sur deux axes (layout qui dicte la taille). En pratique : Grid pour la structure globale de page, Flexbox pour l'alignement des éléments à l'intérieur d'un composant.

### 10 - Responsive design & media queries

```css
/* mobile-first : styles de base = mobile, on surcharge vers le haut */
.carte { padding: 12px; }

@media (min-width: 768px) {
  .carte { padding: 24px; }
}
@media (min-width: 1024px) {
  .carte { padding: 32px; }
}
```

**Erreur fréquente** : écrire en *desktop-first* (`max-width`) par défaut → surcharge de règles à mesure que l'écran rétrécit, plus difficile à maintenir.
**Bonne pratique** : mobile-first (`min-width`) systématique, breakpoints alignés sur le contenu (là où le layout casse visuellement) plutôt que sur des tailles d'appareils figées.

## 4. Concepts intermédiaires 🟡

**Variables CSS (custom properties)** — valeurs réactives, contrairement aux variables Sass qui sont figées à la compilation :

```css
:root {
  --color-primary: #2563eb;
  --spacing-md: 1rem;
}
.btn {
  background: var(--color-primary);
  padding: var(--spacing-md);
}
@media (prefers-color-scheme: dark) {
  :root { --color-primary: #60a5fa; }
}
```

**Pseudo-classes & pseudo-éléments** :

```css
a:hover, a:focus-visible { }
li:first-child, li:last-child, li:nth-child(2n) { }
input:invalid, input:disabled { }
p::first-line { }
.tooltip::before { content: "→"; }
```

**Transitions & animations** :

```css
.btn {
  transition: background-color 200ms ease-in-out;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(4px); }
  to   { opacity: 1; transform: translateY(0); }
}
.modal { animation: fade-in 250ms ease-out; }
```

**Erreur fréquente** : animer `width`/`height`/`top`/`left` (déclenche layout+paint à chaque frame) au lieu de `transform`/`opacity` (compositing seul, accéléré GPU).

**Fonctions modernes** :

```css
width: clamp(200px, 50%, 600px); /* min, préféré, max */
width: min(90%, 800px);
height: calc(100vh - var(--header-height));
```

**Architecture CSS (BEM)** — convention de nommage pour garder une spécificité plate et prévisible à l'échelle :

```css
.carte { }               /* Block */
.carte__titre { }        /* Element */
.carte--mise-en-avant { } /* Modifier */
```

**Bonne pratique** : une seule classe par règle (spécificité `0,1,0` partout), jamais de nesting de sélecteurs qui remonte la spécificité.

**Stacking contexts & z-index** : `z-index` ne compare des éléments qu'au sein du **même contexte d'empilement**. `position` (autre que `static`) + `z-index`, `opacity < 1`, `transform`, ou `filter` créent chacun un nouveau contexte — d'où les "z-index: 9999 qui ne marche pas" quand un ancêtre a déjà créé son propre contexte isolé.

**Debugging CSS** : DevTools → onglet Computed pour voir la valeur finale après cascade, onglet Layout pour visualiser Flexbox/Grid, `outline: 1px solid red` sur `*` pour visualiser rapidement les boîtes.

## 5. Concepts avancés 🟠🔴

**Container queries** — adapter un composant à la taille de **son conteneur**, pas du viewport (essentiel pour des composants réellement réutilisables, ex. une carte qui doit s'adapter qu'elle soit en sidebar étroite ou en grille large) :

```css
.carte-wrapper { container-type: inline-size; container-name: carte; }

@container carte (min-width: 400px) {
  .carte { flex-direction: row; }
}
```

**`@layer` (cascade layers)** — contrôle explicite de l'ordre de priorité entre groupes de règles, indépendamment de leur ordre d'écriture ou de leur spécificité :

```css
@layer reset, base, components, utilities;

@layer components {
  .btn { padding: 8px 16px; }
}
@layer utilities {
  .p-0 { padding: 0; } /* gagne toujours sur .btn même si moins spécifique */
}
```

Résout un problème historique : intégrer une librairie tierce sans que ses styles n'entrent en guerre de spécificité avec les vôtres.

**`:has()`** — le "sélecteur parent" enfin natif :

```css
.carte:has(img) { grid-template-columns: 120px 1fr; }
form:has(input:invalid) .submit { opacity: 0.5; }
```

**Nesting natif CSS** (sans préprocesseur) :

```css
.carte {
  padding: 1rem;
  & .titre { font-weight: 600; }
  &:hover { box-shadow: 0 2px 8px rgb(0 0 0 / 0.1); }
}
```

**Accessibilité liée au CSS** :

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
:focus-visible { outline: 2px solid var(--color-primary); outline-offset: 2px; }
```

`:focus-visible` (plutôt que `:focus`) affiche l'anneau de focus seulement pour la navigation clavier, pas au clic souris — meilleur des deux mondes esthétique/accessibilité.

**Performance : repaint / reflow / layout thrashing** — modifier une propriété géométrique (`width`, `top`...) force un *reflow* (recalcul du layout) qui se propage souvent à tout le document, suivi d'un *repaint*. Lire une propriété géométrique juste après l'avoir écrite (dans une boucle JS) force un recalcul synchrone (*layout thrashing*). `transform`/`opacity` restent sur le thread de compositing et évitent ce coût. `will-change` prévient le navigateur d'une animation à venir pour pré-optimiser, mais coûte de la mémoire GPU si abusé — à réserver aux éléments réellement animés, jamais posé "par précaution" partout.

**Design tokens & theming** — les custom properties permettent un theming complet (dark mode, marques multiples) sans dupliquer les règles, juste en redéfinissant les tokens à la racine ou sur un attribut (`[data-theme="dark"]`).

**Architecture à grande échelle** : ITCSS (couches de spécificité croissante : settings → tools → generic → elements → objects → components → utilities), CSS Modules (scoping automatique par build tool), ou `@layer` natif pour le même objectif sans tooling.

## 6. Commandes / syntaxe à connaître

```css
/* reset de base */
*, *::before, *::after { box-sizing: border-box; margin: 0; }

/* centrage universel */
.center { display: grid; place-items: center; }

/* troncature de texte */
.truncate { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

/* import d'une autre feuille */
@import url("reset.css");

/* media queries courantes */
@media (min-width: 768px) { }
@media (prefers-color-scheme: dark) { }
@media print { }
```

```bash
# outils courants pour travailler le CSS (hors dossier tailwindcss/)
npx sass input.scss output.css --watch   # si Sass utilisé
npx stylelint "**/*.css"                 # linting
npx postcss input.css -o output.css      # pipeline de transformation
```

## 7. Exercices

Énoncés dans `exercices/`, corrections séparées dans `solutions/` — à ne consulter qu'après avoir essayé.

- **Niveau 1 — Bases** : `exercices/niveau-1.md`
- **Niveau 2 — Intermédiaire** : `exercices/niveau-2.md`
- **Niveau 3 — Avancé** : `exercices/niveau-3.md`

## 8. Mini-projet

**Carte produit responsive avec thème clair/sombre**, en CSS pur :

- Grille de 3 cartes (Grid) qui passe à 1 colonne sur mobile (mobile-first)
- Chaque carte utilise Flexbox en interne (image, titre, prix, bouton alignés)
- Variables CSS pour les couleurs, avec bascule dark mode via `prefers-color-scheme` **et** un attribut `data-theme` piloté par un bouton (JS minimal)
- Bouton avec `:hover`, `:focus-visible` et une transition
- `:has()` pour mettre en avant la carte contenant un badge "promo"
- Respect de `prefers-reduced-motion`

Livrable : un unique fichier `index.html` + `style.css`, sans framework ni préprocesseur.

## Checklist

- [ ] Comprendre les fondamentaux (cascade, spécificité, box model)
- [ ] Savoir créer un projet CSS structuré
- [ ] Maîtriser la syntaxe principale (sélecteurs, Flexbox, Grid, responsive)
- [ ] Comprendre les concepts importants (variables, animations, BEM, stacking context)
- [ ] Savoir debugger (DevTools, Computed, Layout)
- [ ] Connaître les bonnes pratiques (mobile-first, spécificité basse, transform/opacity pour animer)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (container queries, `@layer`, `:has()`, nesting natif, perf)

## 10. Ressources

- [MDN — CSS](https://developer.mozilla.org/fr/docs/Web/CSS) — référence officielle, à privilégier systématiquement
- [MDN — Cascade et spécificité](https://developer.mozilla.org/fr/docs/Web/CSS/Cascade)
- [web.dev — Learn CSS](https://web.dev/learn/css) — parcours structuré par Google, très à jour sur les fonctionnalités modernes
- [roadmap.sh — Frontend](https://roadmap.sh/frontend) — CSS y est couvert dans le cadre du parcours frontend complet
- [CSS-Tricks — A Complete Guide to Flexbox / Grid](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) — référence communautaire reconnue pour Flexbox et Grid
