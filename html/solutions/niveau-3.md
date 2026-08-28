# Solutions HTML — Niveau 3

## 3.1 — Refonte sémantique

```html
<body>
  <header>
    <p class="logo">MonSite</p>
    <nav>
      <ul>
        <li><a href="/">Accueil</a></li>
        <li><a href="/produits">Produits</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <article>
      <h1>Nouveau produit disponible</h1>
      <p><time datetime="2026-03-12">12/03/2026</time></p>
      <p>Description du produit...</p>
    </article>

    <aside aria-label="Articles liés">
      <h2>Articles liés</h2>
      <!-- ... -->
    </aside>
  </main>

  <footer>© 2026 MonSite</footer>
</body>
```

Justifications :
- `.top` → `<header>` : c'est l'en-tête de page (logo + nav).
- `.links` → `<nav><ul>` : liste de liens de navigation, pas juste des `<div>` cliquables.
- `.content` global → `<main>` : contenu principal unique de la page.
- `.post` → `<article>` : contenu autonome (un post a du sens seul, ex. dans un flux RSS).
- `.post-title` → `<h1>` (le titre de l'article, unique élément `h1` de la page ici) — si la page avait déjà un `h1` ailleurs, ce serait un `h2`.
- `.post-date` → `<time datetime="...">` : donne une valeur machine-lisible en plus du texte affiché.
- `.sidebar` → `<aside>` : contenu connexe mais secondaire par rapport à l'article.
- `.bottom` → `<footer>` : pied de page.

## 3.2 — Formulaire multi-étapes accessible

Points clés de la structure :

```html
<nav aria-label="Étapes du formulaire">
  <ol>
    <li aria-current="step">Infos perso</li>
    <li>Adresse</li>
    <li>Confirmation</li>
  </ol>
</nav>

<form>
  <fieldset id="step-1">
    <legend>Informations personnelles</legend>
    <!-- champs -->
  </fieldset>

  <fieldset id="step-2" hidden>
    <legend>Adresse</legend>
    <!-- champs -->
  </fieldset>

  <fieldset id="step-3" hidden>
    <legend>Confirmation</legend>
  </fieldset>

  <p aria-live="polite" id="step-status">Étape 1 sur 3</p>
</form>
```

- `aria-current="step"` sur l'étape active dans la nav de progression.
- `aria-live="polite"` sur la zone de statut : annonce automatiquement le changement d'étape sans que l'utilisateur doive naviguer dessus.
- Piège `required` + `hidden` : un champ `required` dans un `<fieldset hidden>` bloque parfois la soumission native même invisible. Solution : retirer/ajouter `required` dynamiquement en JS selon l'étape affichée, plutôt que de le laisser en permanence sur des champs masqués.

## 3.3 — Audit d'accessibilité (exemple de trame de réponse)

Exemple de constats typiques (le contenu réel dépend de la page auditée) :
1. Contraste insuffisant sur un texte gris clair sur fond blanc → correction CSS, hors scope HTML.
2. Saut de niveau de titre (`h1` → `h4` directement) → réorganiser en `h1` → `h2` → `h3`.
3. Plusieurs `<h1>` sur la même page → n'en garder qu'un, requalifier les autres.
4. Champ de recherche sans `<label>` (seulement un `placeholder`) → ajouter `<label for="search">Rechercher</label>` (visuellement masqué en CSS si besoin, mais présent dans le DOM).
5. Bouton "suivant" en `<div>` cliquable non focusable → remplacer par `<button>`.

## 3.4 — Rich snippet SEO (Recipe)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  "name": "Tarte au citron",
  "prepTime": "PT20M",
  "cookTime": "PT35M",
  "recipeIngredient": [
    "3 citrons",
    "150 g de sucre",
    "3 œufs",
    "1 pâte sablée"
  ],
  "recipeInstructions": [
    { "@type": "HowToStep", "text": "Préchauffer le four à 180°C." },
    { "@type": "HowToStep", "text": "Mélanger le jus de citron, le sucre et les œufs." },
    { "@type": "HowToStep", "text": "Verser sur la pâte et cuire 35 minutes." }
  ]
}
</script>
```
