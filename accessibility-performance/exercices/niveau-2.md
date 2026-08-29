# Exercices Accessibility & Performance — Niveau 2 (Intermédiaire)

## Exercice 1 — Image responsive avec formats modernes

Une photo de produit est disponible en trois résolutions (400px, 800px, 1200px de large) et trois formats (AVIF, WebP, JPEG). Écris le balisage `<picture>` qui sert le format le plus performant supporté par le navigateur, avec une image responsive selon la largeur d'écran.

## Exercice 2 — Live region

Un formulaire affiche un message de succès après soumission via une mise à jour du DOM en JavaScript (sans rechargement de page). Un utilisateur de lecteur d'écran ne l'entend pas. Corrige le balisage du conteneur de message pour qu'il soit annoncé automatiquement.

## Exercice 3 — Font-display

Une police custom est chargée via `@font-face` sans propriété `font-display`. Explique le problème que cela cause au chargement de la page, puis corrige la déclaration.

## Exercice 4 — Niveaux de conformité WCAG

Explique en 2-3 phrases la différence entre les niveaux A, AA et AAA du WCAG, et quel niveau est généralement visé comme standard professionnel.

## Exercice 5 — Lazy loading

Une page produit affiche 30 images de produits recommandés en bas de page, jamais visibles au chargement initial pour la majorité des visiteurs. Explique comment réduire leur impact sur le temps de chargement initial, avec le code correspondant.
