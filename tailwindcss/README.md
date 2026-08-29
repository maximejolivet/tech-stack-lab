# Tailwind CSS

## 1. Introduction

Tailwind CSS est un framework CSS **utility-first** : au lieu d'écrire des classes sémantiques (`.card`, `.btn-primary`) puis leur CSS associé, on compose directement le style dans le HTML via de petites classes utilitaires à responsabilité unique (`flex`, `p-4`, `text-lg`). Ce dossier suppose le CSS acquis (voir [`../css/`](../css/)) et se concentre sur ce que Tailwind change dans la façon d'écrire du style.

**À quoi sert-il ?**
- Styliser rapidement une interface sans quitter le HTML/JSX pour aller éditer un fichier CSS séparé.
- Garder un design cohérent en contraignant les choix (espacements, couleurs, tailles) à une échelle prédéfinie plutôt que des valeurs arbitraires.
- Éviter la croissance incontrôlée d'une feuille de style au fil du temps (le CSS "mort" ou dupliqué est un problème classique en maintenance longue durée).

**Où se situe-t-il dans une architecture web ?** Couche présentation, côté client — s'intègre à n'importe quel framework front ([`../react/`](../react/), [`../vuejs/`](../vuejs/), [`../angular/`](../angular/)) ou à du HTML statique, via un plugin de build (Vite, PostCSS) qui scanne le code source et ne génère que le CSS réellement utilisé.

**Avantages** : pas de changement de contexte entre HTML et CSS, design system cohérent par construction (échelle d'espacement/couleurs partagée), bundle CSS final très petit grâce au moteur JIT (voir avancé), aucune bataille de spécificité CSS/cascade à gérer.
**Limites** : le HTML devient verbeux (beaucoup de classes par élément), courbe d'apprentissage du nommage des utilitaires, nécessite une étape de build (pas utilisable en `<link>` CDN simple en production).

## 2. Prérequis

- CSS solide : box model, Flexbox, Grid, spécificité, media queries (voir [`../css/`](../css/)).
- Idéalement un projet existant (React/Vue/Angular ou HTML+Vite) pour brancher l'outillage de build.
- Node.js et npm installés.

## 3. Rappel des bases 🟢

### 01 - Installation

**Explication** — Tailwind s'installe comme dépendance de build et génère son CSS à partir d'un fichier de config qui scanne le code source à la recherche des classes utilisées.

```bash
npm install -D tailwindcss @tailwindcss/postcss postcss
```

```css
/* src/index.css */
@import "tailwindcss";
```

**Bonne pratique** : ne jamais utiliser le CDN `<script src="https://cdn.tailwindcss.com">` en production — il charge le moteur complet côté navigateur au lieu d'un CSS pré-généré et minifié, ce qui est nettement plus lourd et plus lent.

### 02 - Classes utilitaires de base

**Explication** — Chaque classe correspond à une propriété CSS unique, avec une valeur issue d'une échelle prédéfinie (espacement en multiples de `0.25rem`, palette de couleurs à nuances numérotées).

```html
<div class="flex items-center gap-4 p-6 bg-white rounded-lg shadow-md">
  <h2 class="text-xl font-bold text-gray-900">Titre</h2>
</div>
```

**Erreur fréquente** : mélanger l'échelle Tailwind avec des valeurs CSS arbitraires non nécessaires (`p-[13px]` au lieu de `p-3`/`p-4`) — casse la cohérence visuelle que l'échelle est censée garantir. Réserver les valeurs arbitraires (`w-[327px]`) aux cas où aucune valeur de l'échelle ne convient réellement (ex. contrainte pixel-perfect d'une maquette).

### 03 - Layout : Flexbox et Grid

**Explication** — Classes directement mappées aux propriétés Flexbox/Grid.

```html
<div class="grid grid-cols-3 gap-4">
  <div class="col-span-2">Contenu principal</div>
  <div>Sidebar</div>
</div>
```

**Bonne pratique** : préférer `gap-*` à des marges manuelles entre éléments d'un conteneur flex/grid — évite les marges asymétriques sur le premier/dernier élément.

### 04 - Responsive design (mobile-first)

**Explication** — Les préfixes responsifs (`sm:`, `md:`, `lg:`, `xl:`, `2xl:`) s'appliquent **à partir de** ce breakpoint (mobile-first) : une classe sans préfixe s'applique à toutes les tailles, un préfixe la surcharge à partir du seuil donné.

```html
<div class="text-sm md:text-base lg:text-lg">
  Texte qui grossit progressivement
</div>
```

**Erreur fréquente** : penser en "desktop-first" et utiliser un préfixe pour cibler le mobile — Tailwind est mobile-first, donc le style sans préfixe est la base mobile, et les préfixes ajoutent des styles pour les écrans **plus grands**, jamais l'inverse.

### 05 - États : hover, focus, etc.

**Explication** — Les variantes d'état préfixent la classe utilitaire directement, sans média query ni pseudo-classe séparée à écrire.

```html
<button class="bg-blue-600 hover:bg-blue-700 focus:ring-2 focus:ring-blue-400 disabled:opacity-50">
  Valider
</button>
```

**Bonne pratique** : toujours prévoir un état `focus:` visible sur les éléments interactifs (accessibilité clavier), pas seulement `hover:` — voir [`../accessibility-performance/`](../accessibility-performance/).

### 06 - Couleurs et thème

**Explication** — Chaque couleur a une échelle de nuances numérotées de 50 (très clair) à 950 (très foncé), permettant des variations cohérentes (`bg-blue-50` pour un fond léger, `text-blue-900` pour un texte contrasté de la même teinte).

```html
<div class="bg-red-50 text-red-800 border border-red-200 p-4 rounded">
  Message d'erreur
</div>
```

## 4. Concepts intermédiaires 🟡

- **Personnalisation du thème** : le fichier de config (ou directement en CSS via `@theme` dans les versions récentes) redéfinit l'échelle de couleurs, d'espacement, de polices pour correspondre à une charte graphique, plutôt que de se limiter aux valeurs par défaut.

```css
@theme {
  --color-brand: oklch(0.6 0.2 250);
  --font-display: "Inter", sans-serif;
}
```

- **`@apply`** : permet de regrouper plusieurs utilitaires sous une classe CSS nommée, pour les cas où la répétition devient un vrai problème de maintenance.

```css
.btn-primary {
  @apply bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700;
}
```

**Erreur fréquente / piège** : abuser de `@apply` pour recréer un système de classes sémantiques façon Bootstrap — cela annule l'intérêt principal de Tailwind (co-localisation du style avec le markup, pas de fichier CSS à maintenir en parallèle) et réintroduit les problèmes que le framework cherche à éviter.

**Bonne pratique** : dans un framework à composants (React/Vue), préférer l'extraction en **composant** (`<Button>`) à l'extraction en classe CSS via `@apply` — le composant encapsule aussi le comportement, pas seulement le style.

- **Dark mode** : variante `dark:` activée par préférence système ou classe manuelle sur `<html>`, selon la stratégie configurée.

```html
<div class="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
```

- **Variantes combinées et group/peer** : `group-hover:`, `peer-checked:` permettent de styliser un élément en fonction de l'état d'un élément parent (`group`) ou voisin (`peer`), sans JavaScript.

```html
<label class="group flex items-center">
  <input type="checkbox" class="peer" />
  <span class="peer-checked:text-green-600 group-hover:underline">Option</span>
</label>
```

- **Composants réutilisables (approche framework)** : dans React/Vue, factoriser un ensemble de classes répété dans un composant plutôt que de le copier-coller — c'est l'unité de réutilisation recommandée par-dessus Tailwind, pas les classes CSS custom.

## 5. Concepts avancés 🟠🔴

- **Moteur JIT (Just-In-Time)** : Tailwind ne génère pas un fichier CSS statique exhaustif ; il scanne le code source à chaque build et ne produit **que** les classes réellement utilisées, y compris les valeurs arbitraires (`top-[117px]`) ou les variantes combinées complexes — d'où un CSS final très léger même avec un thème très large.

**Erreur fréquente** : construire dynamiquement un nom de classe par concaténation de chaînes (`` `text-${color}-500` ``) — le scanner JIT analyse le code source de façon statique et ne peut pas résoudre une classe qui n'apparaît nulle part sous sa forme complète littérale, elle sera donc absente du CSS généré. Toujours écrire la classe complète, éventuellement choisie parmi un objet de mapping explicite.

- **Content configuration** : la liste des fichiers à scanner (`content: [...]`) doit couvrir précisément tous les fichiers source utilisant des classes Tailwind — un chemin oublié produit des classes manquantes en production sans erreur visible en dev si le cache local les contient encore.
- **Plugins officiels** : `@tailwindcss/forms` (reset cohérent des champs de formulaire), `@tailwindcss/typography` (classe `prose` pour du contenu riche généré, ex. Markdown/CMS), `@tailwindcss/container-queries`.
- **Container queries** : styliser un élément selon la taille de son **conteneur** parent plutôt que celle du viewport (`@container`, `@sm:`) — complète les media queries classiques pour des composants réellement réutilisables dans des contextes de largeur variable.
- **Performance de build** : le moteur JIT recalcule vite en développement (incrémental), mais un `content` mal scopé (ex. incluant `node_modules`) peut ralentir significativement les builds — toujours restreindre le scan aux fichiers source du projet.
- **Cohérence design system à l'échelle d'une équipe** : sur un gros projet, documenter/contraindre les valeurs autorisées (via le thème personnalisé) évite que chaque développeur réintroduise des valeurs arbitraires incohérentes ; des outils de lint (`eslint-plugin-tailwindcss`) peuvent forcer l'ordre des classes et détecter les classes inconnues.

## 6. Commandes / syntaxe à connaître

```bash
npm install -D tailwindcss @tailwindcss/postcss postcss   # installation
npx tailwindcss -i src/input.css -o dist/output.css --watch   # build CLI standalone (hors framework)
```

```html
<div class="flex flex-col md:flex-row items-center justify-between gap-4 p-6 bg-white dark:bg-gray-900 rounded-xl shadow hover:shadow-lg transition">
  <!-- layout responsive + dark mode + état hover, tout en utilitaires -->
</div>
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Carte de profil responsive avec dark mode**

Construire en HTML + Tailwind (sans framework JS requis) :
- Une carte (`card`) avec avatar, nom, bio, et 3 boutons d'action, centrée verticalement sur mobile et en layout horizontal à partir de `md:`.
- Un support complet du dark mode (`dark:`) sur tous les éléments (fond, texte, bordures).
- Un bouton "Suivre" avec des états `hover:`, `focus:`, `disabled:` visuellement distincts.
- Un badge de statut utilisant `peer`/`group` pour changer d'apparence selon l'état d'une checkbox cachée ("en ligne"/"hors ligne").
- Bonus : extraire les classes répétées des 3 boutons dans un composant réutilisable si le projet est branché à React/Vue, plutôt que via `@apply`.

## Checklist

- [ ] Comprendre les fondamentaux (utility-first, échelle d'espacement/couleurs, installation)
- [ ] Savoir configurer Tailwind dans un projet (build, content/scan)
- [ ] Maîtriser la syntaxe principale (layout, responsive mobile-first, états hover/focus)
- [ ] Comprendre les concepts importants (`@apply`, dark mode, group/peer)
- [ ] Savoir debugger (classes manquantes en prod, mauvaise config `content`)
- [ ] Connaître les bonnes pratiques (composant plutôt que `@apply`, éviter les classes construites dynamiquement)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (moteur JIT, container queries, plugins officiels)

## 10. Ressources

- [Documentation officielle Tailwind CSS](https://tailwindcss.com/docs) — référence complète des utilitaires.
- [Tailwind UI](https://tailwindui.com/) — composants officiels payants, utile pour voir des patterns idiomatiques.
- [Headless UI](https://headlessui.com/) — composants accessibles sans style, pensés pour être stylés avec Tailwind.
- Il n'existe pas de roadmap.sh dédiée à Tailwind CSS à ce jour ; voir [roadmap.sh — Frontend](https://roadmap.sh/frontend) pour le contexte plus large, et [`../css/`](../css/) pour les fondamentaux CSS sous-jacents.
