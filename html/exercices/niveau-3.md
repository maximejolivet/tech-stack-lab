# Exercices HTML — Niveau 3 (Avancé)

## Exercice 3.1 — Refonte sémantique

Voici le squelette (simplifié) d'une page produite par un développeur junior. Refactorise-la entièrement en HTML sémantique et accessible, sans changer le contenu textuel.

```html
<div id="page">
  <div class="top">
    <div class="logo">MonSite</div>
    <div class="links">
      <div>Accueil</div>
      <div>Produits</div>
      <div>Contact</div>
    </div>
  </div>
  <div class="content">
    <div class="post">
      <div class="post-title">Nouveau produit disponible</div>
      <div class="post-date">12/03/2026</div>
      <div class="post-body">Description du produit...</div>
    </div>
    <div class="sidebar">
      <div>Articles liés</div>
    </div>
  </div>
  <div class="bottom">© 2026 MonSite</div>
</div>
```

Identifie et justifie chaque changement de balise (pourquoi celle-ci et pas une autre).

## Exercice 3.2 — Formulaire multi-étapes accessible

Conçois la structure HTML d'un formulaire d'inscription en 3 étapes (Infos perso → Adresse → Confirmation), où une seule étape est visible à la fois. Réfléchis à :
- comment annoncer le changement d'étape à un lecteur d'écran (`aria-live` ou équivalent),
- comment gérer la validation native sur des champs actuellement masqués (piège fréquent : `required` sur un champ `hidden` bloque parfois la soumission),
- la structure de progression (`<nav aria-label="Étapes">` avec l'étape courante indiquée).

## Exercice 3.3 — Audit d'accessibilité

Prends une page HTML existante de ton choix (un vieux projet, ou une page publique). Audite-la avec l'outil "Accessibility" des DevTools Chrome/Firefox (ou l'extension axe DevTools) et liste :
- 3 problèmes de contraste ou de structure de titres,
- 1 problème de formulaire (label manquant, `alt` absent...),
- 1 problème de navigation clavier.

Propose une correction HTML pour chacun (pas de correction CSS).

## Exercice 3.4 — Rich snippet SEO

Pour une page de recette de cuisine, écris le balisage JSON-LD `schema.org/Recipe` (nom, temps de préparation, ingrédients, instructions) qui permettrait à Google d'afficher un rich snippet dans les résultats de recherche.
