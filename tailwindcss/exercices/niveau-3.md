# Exercices Tailwind CSS — Niveau 3 (Avancé)

## Exercice 1 — Piège du JIT

Voici du code React :

```tsx
function Badge({ color }: { color: "red" | "green" | "blue" }) {
  return <span className={`bg-${color}-500 text-white px-2 py-1 rounded`}>Statut</span>;
}
```

Explique pourquoi ce code ne fonctionnera pas correctement une fois buildé en production, puis corrige-le en écrivant les classes de façon détectable par le moteur JIT.

## Exercice 2 — Configuration content

Un projet Vite + React a sa configuration Tailwind qui scanne uniquement `./src/**/*.{js,jsx}`, mais le projet utilise des fichiers `.tsx`. Identifie le problème et corrige la configuration.

## Exercice 3 — Container queries

Un composant `Card` doit afficher son contenu en colonne quand son conteneur fait moins de 400px de large, et en ligne au-delà — indépendamment de la taille du viewport global. Explique pourquoi une media query classique (`md:flex-row`) ne résout pas ce problème, et écris la solution avec `@container`.

## Exercice 4 — Plugin typography

Un article de blog dont le contenu HTML est généré depuis un Markdown externe doit être stylé (titres, listes, citations) sans écrire de classes utilitaires sur chaque balise générée. Quel plugin officiel utiliser, et quelle classe appliquer sur le conteneur ?

## Exercice 5 — Composant vs @apply, arbitrage d'équipe

Une équipe de 6 développeurs React constate que le style d'une carte produit (image, titre, prix, bouton) est dupliqué dans 12 fichiers différents avec de légères incohérences. Propose et justifie en 3-4 phrases la solution à privilégier entre : (a) créer une classe `.product-card` via `@apply`, (b) créer un composant `<ProductCard>` réutilisable.
