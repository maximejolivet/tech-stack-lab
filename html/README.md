# HTML

## 1. Introduction

HTML (HyperText Markup Language) décrit la **structure et la sémantique** d'un document web : ce qu'est chaque bloc de contenu (un titre, une navigation, un formulaire, un article), pas son apparence.

- **À quoi ça sert** : donner un sens machine-lisible au contenu, pour que le navigateur, les lecteurs d'écran, les moteurs de recherche et les autres développeurs comprennent la structure sans avoir à deviner.
- **Quand l'utiliser** : sur tout projet web, systématiquement. Même dans une SPA React/Vue, le HTML final rendu doit rester sémantique.
- **Problèmes résolus** : sans sémantique, un site n'est ni accessible (lecteurs d'écran), ni bien référencé (SEO), ni maintenable (un `<div>` ne dit rien sur son rôle).
- **Où il se situe dans l'architecture web** : c'est la couche la plus basse du front — le CSS stylise le HTML, le JS le manipule (DOM). Un mauvais HTML de base complique tout le reste.
- **Avantages** : universel, interprété nativement par tous les navigateurs, base de l'accessibilité et du SEO.
- **Limites** : ne gère ni le style (→ [`css/`](../css/)) ni le comportement dynamique (→ [`javascript/`](../javascript/)). Le HTML seul ne fait "rien" d'interactif.

> Ce dossier couvre la structure et la sémantique du document. Le style (mise en page, couleurs, responsive) est traité dans [`css/`](../css/) et [`tailwindcss/`](../tailwindcss/).

## 2. Prérequis

Aucun prérequis technique. Une notion de base du fonctionnement client-serveur (requête → réponse HTML → rendu navigateur) aide à situer le contexte.

## 3. Rappel des bases 🟢

### 3.1 Structure du document

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Titre de la page</title>
</head>
<body>
  <!-- contenu visible -->
</body>
</html>
```

- `<!DOCTYPE html>` force le mode standard (évite le "quirks mode" qui casse le rendu CSS).
- `lang` sur `<html>` : indispensable pour l'accessibilité (prononciation correcte par les lecteurs d'écran) et le SEO.
- `<meta charset="UTF-8">` doit être la **première** balise dans `<head>` (dans les 1024 premiers octets) sinon risque de mauvais encodage.
- `viewport` : sans lui, un mobile affiche la page zoomée comme sur desktop.

**Erreur fréquente** : oublier `lang` ou le mettre sur `<body>` au lieu de `<html>`.
**Bonne pratique** : toujours valider la structure avec le [validateur W3C](https://validator.w3.org/) sur un projet important.

### 3.2 Sémantique : les balises structurantes

```html
<body>
  <header>
    <nav>...</nav>
  </header>
  <main>
    <article>
      <section>...</section>
    </article>
    <aside>...</aside>
  </main>
  <footer>...</footer>
</body>
```

| Balise | Rôle | Cas d'usage |
|---|---|---|
| `<header>` | En-tête (page ou section) | Logo, titre, nav — répétable par section |
| `<nav>` | Bloc de navigation principal | Menu, fil d'Ariane, pagination |
| `<main>` | Contenu principal unique de la page | Une seule fois par page |
| `<article>` | Contenu autonome, redistribuable seul | Un post de blog, un commentaire |
| `<section>` | Regroupement thématique avec un titre | Un chapitre, une partie de page |
| `<aside>` | Contenu connexe, secondaire | Sidebar, encart pub, liens liés |
| `<footer>` | Pied (page ou section) | Copyright, liens légaux |

**Différence `<article>` vs `<section>`** : `<article>` doit avoir du sens hors contexte (un flux RSS pourrait l'extraire tel quel). `<section>` sert juste à regrouper visuellement/thématiquement.

**Erreur fréquente** : tout mettre en `<div>` ("divitis") — perte totale de sens pour l'accessibilité et le SEO. Ou l'excès inverse : `<section>` partout sans titre associé (chaque `<section>` devrait avoir un `<h1>`-`<h6>`).

**Bonne pratique** : se demander "quel est le *rôle* de ce bloc ?" avant de choisir la balise, et se poser la question — est-ce que je peux nommer ce bloc autrement que `<div>` ?

### 3.3 Titres et hiérarchie

```html
<h1>Titre de la page</h1>
<h2>Section</h2>
<h3>Sous-section</h3>
```

- Un seul `<h1>` par page (sujet principal), hiérarchie sans saut de niveau (`h1` → `h3` sans `h2` = mauvaise pratique).
- Les lecteurs d'écran permettent de naviguer *par titres* : une hiérarchie cassée casse la navigation.

**Erreur fréquente** : choisir un niveau de titre pour son rendu visuel (taille) plutôt que sa position sémantique — c'est le rôle du CSS de gérer la taille.

### 3.4 Texte et listes

```html
<p>Paragraphe. <strong>Important</strong>, <em>emphase</em>.</p>

<ul>
  <li>Élément non ordonné</li>
</ul>

<ol>
  <li>Élément ordonné</li>
</ol>

<dl>
  <dt>Terme</dt>
  <dd>Définition</dd>
</dl>
```

- `<strong>` = importance sémantique (souvent en gras) ≠ `<b>` (gras purement visuel, sans sens).
- `<em>` = emphase sémantique (souvent en italique) ≠ `<i>` (italique visuel).

**Bonne pratique** : préférer `<strong>`/`<em>` à `<b>`/`<i>` sauf cas très spécifique (ex. nom de bateau en `<i>` par convention typographique, sans emphase).

### 3.5 Liens et images

```html
<a href="/contact" target="_blank" rel="noopener noreferrer">Contact</a>

<img src="photo.jpg" alt="Description de l'image pour l'accessibilité" width="800" height="600" loading="lazy">
```

- `alt` : **obligatoire**. Vide (`alt=""`) si l'image est purement décorative (elle est alors ignorée par le lecteur d'écran), sinon description du contenu/fonction de l'image.
- `target="_blank"` sans `rel="noopener"` expose à une faille de sécurité : la page ouverte peut accéder à `window.opener` et rediriger l'onglet parent (*reverse tabnabbing*).
- `width`/`height` : évitent le *layout shift* (CLS) pendant le chargement, même si la taille finale est gérée en CSS.

**Erreur fréquente** : `alt="image"` ou `alt="photo123.jpg"` — ne décrit rien, inutile voire nuisible.
**Bonne pratique** : `alt` décrit la *fonction* de l'image dans son contexte (pas juste son contenu visuel). Une icône de loupe qui déclenche une recherche : `alt="Rechercher"`, pas `alt="icône loupe"`.

### 3.6 Formulaires

```html
<form action="/submit" method="post">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required autocomplete="email">

  <label for="msg">Message</label>
  <textarea id="msg" name="msg" required minlength="10"></textarea>

  <button type="submit">Envoyer</button>
</form>
```

- `<label for="id">` lié à l'`id` du champ : indispensable pour l'accessibilité (le lecteur d'écran annonce le label au focus) **et** l'UX (cliquer sur le label focus le champ).
- Types d'`<input>` sémantiques (`email`, `tel`, `number`, `date`, `url`...) : déclenchent le bon clavier mobile et une validation native basique.
- Validation native : `required`, `minlength`, `maxlength`, `pattern`, `min`, `max` — validée par le navigateur **avant** tout JS, mais **jamais suffisante côté sécurité** (toujours revalider côté serveur).
- `autocomplete` : accélère la saisie et améliore l'UX (valeurs standard : `email`, `name`, `tel`, `new-password`, etc.).

**Erreur fréquente** : `<div onclick="submit()">Envoyer</div>` à la place d'un vrai `<button>` — perte du focus clavier, de l'activation par `Entrée`/`Espace`, et du rôle sémantique.
**Bonne pratique** : toujours utiliser `<form>` + `<button type="submit">`, même pour un formulaire géré entièrement en JS/fetch — la sémantique et l'accessibilité clavier restent gratuites.

### 3.7 Tableaux

```html
<table>
  <caption>Ventes par trimestre</caption>
  <thead>
    <tr><th scope="col">Trimestre</th><th scope="col">Total</th></tr>
  </thead>
  <tbody>
    <tr><th scope="row">Q1</th><td>12 000 €</td></tr>
  </tbody>
</table>
```

- Un tableau HTML sert à des **données tabulaires**, jamais à la mise en page (pratique des années 2000, obsolète depuis Flexbox/Grid).
- `<th scope="col|row">` : indique aux lecteurs d'écran à quelle ligne/colonne une cellule appartient.

**Erreur fréquente** : utiliser `<table>` pour la mise en page générale d'une page.

## 4. Concepts intermédiaires 🟡

### 4.1 `<head>`, meta tags et SEO de base

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Titre unique et descriptif — Nom du site</title>
  <meta name="description" content="Résumé de la page en ~155 caractères pour les résultats de recherche.">
  <link rel="canonical" href="https://example.com/page">

  <!-- Open Graph (partage réseaux sociaux) -->
  <meta property="og:title" content="Titre pour le partage">
  <meta property="og:description" content="Description pour le partage">
  <meta property="og:image" content="https://example.com/image.jpg">

  <link rel="icon" href="/favicon.ico">
</head>
```

- `<title>` : premier facteur SEO on-page, doit être unique par page.
- `rel="canonical"` : évite le contenu dupliqué aux yeux des moteurs de recherche (ex. `?page=1` vs page sans query string).
- Open Graph : sans ça, un lien partagé sur Slack/LinkedIn/X n'a pas de preview correcte.

### 4.2 Accessibilité : landmarks et ARIA de base

Les balises sémantiques (`<nav>`, `<main>`...) créent déjà des **landmarks ARIA implicites**. ARIA ne sert qu'à combler ce que le HTML natif ne peut pas exprimer.

```html
<button aria-expanded="false" aria-controls="menu">Menu</button>
<ul id="menu" hidden>...</ul>

<div role="alert">Une erreur est survenue.</div>

<button aria-label="Fermer" class="icon-close"></button>
```

- **Règle d'or ARIA** : "No ARIA is better than bad ARIA". Toujours préférer une balise sémantique native à un `role` ARIA sur un `<div>`. `<button>` natif > `<div role="button" tabindex="0">`.
- `aria-label` : donne un nom accessible quand il n'y a pas de texte visible (bouton icône seul).
- `aria-expanded`/`aria-controls` : état d'un composant interactif (accordéon, menu déroulant) que le CSS/JS seul ne communique pas au lecteur d'écran.
- `role="alert"` : annonce automatiquement le contenu injecté dynamiquement (message d'erreur, notification) sans que l'utilisateur ait à naviguer dessus.

**Erreur fréquente** : ajouter `role="button"` sur un `<div>` sans gérer le focus (`tabindex="0"`) ni les touches clavier (`Enter`/`Espace`) — ARIA change ce qui est *annoncé*, pas le comportement, qu'il faut recoder soi-même en JS.
**Bonne pratique** : tester au clavier (Tab, Entrée, Échap) avant de considérer un composant accessible.

### 4.3 Éléments interactifs natifs modernes

```html
<details>
  <summary>Voir plus</summary>
  <p>Contenu masqué par défaut, sans JS.</p>
</details>

<dialog id="modal">
  <p>Contenu de la modale</p>
  <button onclick="document.getElementById('modal').close()">Fermer</button>
</dialog>
```

- `<details>`/`<summary>` : accordéon natif, accessible clavier par défaut, zéro JS.
- `<dialog>` : modale native (`.showModal()` en JS) qui gère le *focus trap* et la touche `Échap` nativement — remplace des années de librairies JS pour ça.

**Bonne pratique** : avant d'installer une lib JS pour un accordéon/modale simple, vérifier si `<details>`/`<dialog>` suffit.

### 4.4 Templates natifs et contenu dynamique

```html
<template id="row-template">
  <tr><td class="name"></td><td class="value"></td></tr>
</template>
```

```js
const clone = document.getElementById('row-template').content.cloneNode(true);
```

- `<template>` : contenu **inerte** (non rendu, scripts non exécutés, images non chargées) tant qu'il n'est pas cloné en JS — utile pour des listes générées côté client sans dépendre d'un framework.

### 4.5 Médias et performance

```html
<img src="photo.jpg" alt="..." loading="lazy" decoding="async">

<picture>
  <source srcset="photo.avif" type="image/avif">
  <source srcset="photo.webp" type="image/webp">
  <img src="photo.jpg" alt="...">
</picture>
```

- `loading="lazy"` : ne charge l'image que quand elle approche du viewport — gain net sur les pages longues. À ne **pas** mettre sur l'image visible au premier écran (LCP), qui doit charger en priorité.
- `<picture>` : sert le format le plus léger supporté par le navigateur (AVIF > WebP > JPEG en fallback).

## 5. Concepts avancés 🟠🔴

### 5.1 Web Components (custom elements)

```html
<script>
  class MyCounter extends HTMLElement {
    connectedCallback() {
      this.innerHTML = `<button>Compteur: <span>0</span></button>`;
    }
  }
  customElements.define('my-counter', MyCounter);
</script>

<my-counter></my-counter>
```

- Permet de créer des balises HTML personnalisées réutilisables, indépendantes de tout framework — base technique sur laquelle s'appuient des libs comme Lit.
- **Shadow DOM** : encapsule le style et le markup d'un composant pour éviter les collisions CSS globales.

### 5.2 Microdata / JSON-LD (SEO avancé)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Titre de l'article",
  "author": { "@type": "Person", "name": "Maxime" }
}
</script>
```

- Balisage structuré (schema.org) : permet aux moteurs de recherche d'afficher des *rich snippets* (étoiles, prix, FAQ dépliable) dans les résultats.

### 5.3 Rendu, hydratation et SSR/CSR

- **CSR** (Client-Side Rendering) : le serveur renvoie un HTML quasi vide, le JS construit le DOM (SPA classique React/Vue).
- **SSR** (Server-Side Rendering) : le serveur renvoie le HTML déjà rempli, puis le JS "hydrate" (attache les event listeners) — meilleur *First Contentful Paint* et SEO.
- Comprendre cette distinction est indispensable avant d'aborder Nuxt/Next côté framework — le HTML initial envoyé par le serveur n'est pas toujours celui vu "à la fin" par l'utilisateur.

### 5.4 Sécurité liée au HTML

- **XSS via innerHTML** : injecter du HTML non échappé venant d'un utilisateur (`el.innerHTML = userInput`) permet l'exécution de `<script>` arbitraire. Toujours échapper ou utiliser `textContent` pour du texte brut.
- **`target="_blank"` sans `rel="noopener"`** (vu en 3.5) : faille classique encore très répandue.
- **Formulaires et CSRF** : un `<form method="post">` sans jeton anti-CSRF est vulnérable à des soumissions forgées depuis un autre site (détail traité dans [`security/`](../security/)).

### 5.5 Performance et Core Web Vitals liés au HTML

- Ordre de chargement des ressources dans `<head>` (CSS bloquant le rendu, `<script defer>` vs `async` vs synchrone) impacte directement le *Largest Contentful Paint* (LCP).
- `width`/`height` sur `<img>`/`<video>` évitent le *Cumulative Layout Shift* (CLS).
- Approfondi dans [`accessibility-performance/`](../accessibility-performance/).

## 6. Commandes / syntaxe à connaître

```html
<!-- Commentaire HTML -->

<!-- Attributs booléens : présence = true, pas besoin de valeur -->
<input type="checkbox" checked disabled>

<!-- Entités HTML utiles -->
&amp; &lt; &gt; &nbsp; &copy;

<!-- Attributs data-* : stocker des données custom, lisibles en JS via dataset -->
<div data-user-id="42"></div>
<!-- js: el.dataset.userId -->

<!-- Auto-fermeture des balises "void" (pas de balise fermante) -->
<br> <hr> <img> <input> <meta> <link>
```

```bash
# Validation W3C en ligne de commande (via npx, sans installation globale)
npx html-validate index.html

# Serveur statique local rapide pour tester du HTML pur
npx serve .
```

## 7. Exercices

Énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/).

- **Niveau 1** — [exercices/niveau-1.md](exercices/niveau-1.md) : structure de base, sémantique simple.
- **Niveau 2** — [exercices/niveau-2.md](exercices/niveau-2.md) : formulaire complet, tableau, accessibilité.
- **Niveau 3** — [exercices/niveau-3.md](exercices/niveau-3.md) : refonte sémantique d'une page mal structurée, cas pro.

## 8. Mini-projet

**CV / page "About me" en HTML pur, sémantiquement irréprochable.**

Contraintes :
- Une seule page, zéro CSS/JS (le style viendra en pratiquant `css/`).
- Doit passer le [validateur W3C](https://validator.w3.org/) sans erreur.
- Doit être 100% navigable au clavier et cohérent pour un lecteur d'écran (tester avec VoiceOver sur Mac : `Cmd+F5`).
- Doit contenir : un header avec nav, une section expérience (liste), une section compétences, un formulaire de contact fonctionnel (validation native), un footer.
- Bonus : ajouter le JSON-LD `schema.org/Person`.

## Checklist

- [ ] Comprendre les fondamentaux (structure, sémantique, hiérarchie de titres)
- [ ] Savoir créer un document HTML valide depuis zéro
- [ ] Maîtriser la syntaxe principale (balises structurantes, formulaires, tableaux, médias)
- [ ] Comprendre les concepts importants (head/SEO, accessibilité/ARIA, éléments interactifs natifs)
- [ ] Savoir debugger (validateur W3C, inspecteur d'accessibilité du navigateur)
- [ ] Connaître les bonnes pratiques (sémantique avant `<div>`, `alt` toujours pertinent, `label` lié aux champs)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Web Components, SSR/CSR, sécurité, performance)

## 10. Ressources

- [MDN — HTML](https://developer.mozilla.org/fr/docs/Web/HTML) — référence officielle, la plus fiable au quotidien.
- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/) — la spécification de référence (source de vérité ultime).
- [W3C Markup Validator](https://validator.w3.org/) — outil de validation.
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) — patterns d'accessibilité de référence pour les composants interactifs.
- [web.dev — Learn HTML](https://web.dev/learn/html/) — Google, orienté bonnes pratiques modernes et performance.
- [roadmap.sh — Frontend](https://roadmap.sh/frontend) — HTML y est couvert dans le cadre du parcours frontend complet.
