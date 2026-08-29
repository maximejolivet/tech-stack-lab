# Solutions — Niveau 3 (Avancé)

## Exercice 1

Le moteur JIT scanne le code source **statiquement** à la recherche de noms de classes complets. `` `bg-${color}-500` `` ne contient jamais la chaîne littérale `bg-red-500` dans le fichier : elle est construite dynamiquement à l'exécution, donc invisible au scanner, et ces classes ne seront tout simplement pas générées dans le CSS final.

```tsx
const COLOR_CLASSES: Record<"red" | "green" | "blue", string> = {
  red: "bg-red-500",
  green: "bg-green-500",
  blue: "bg-blue-500",
};

function Badge({ color }: { color: "red" | "green" | "blue" }) {
  return <span className={`${COLOR_CLASSES[color]} text-white px-2 py-1 rounded`}>Statut</span>;
}
```

Les classes complètes apparaissent désormais littéralement dans le fichier source, donc le scanner JIT les détecte et les génère.

## Exercice 2

Le `content`/scan ne couvre pas les fichiers `.tsx`, donc toute classe utilisée uniquement dans un composant `.tsx` est absente du CSS généré.

```js
// tailwind.config.js (ou équivalent @source en CSS)
export default {
  content: ["./index.html", "./src/**/*.{js,jsx,ts,tsx}"],
};
```

## Exercice 3

Une media query (`md:flex-row`) réagit à la taille du **viewport**, pas à celle du conteneur parent du composant — si `Card` est placé dans une sidebar étroite sur un grand écran, `md:` serait quand même actif alors que l'espace réel disponible est petit. Les container queries résolvent ce problème en réagissant à la taille réelle du conteneur.

```html
<div class="@container">
  <div class="flex flex-col @sm:flex-row gap-4">
    <img class="w-full @sm:w-32" src="..." />
    <div>Contenu de la carte</div>
  </div>
</div>
```

## Exercice 4

Le plugin `@tailwindcss/typography`, avec la classe `prose` (et ses variantes `prose-lg`, `dark:prose-invert`) appliquée sur le conteneur du contenu HTML généré :

```html
<article class="prose dark:prose-invert">
  <!-- HTML généré depuis Markdown, stylé automatiquement -->
</article>
```

## Exercice 5

Le composant `<ProductCard>` est la solution à privilégier. `@apply` ne résoudrait que la duplication du **CSS**, mais le problème réel est la duplication de **structure** (balises image/titre/prix/bouton) répétée dans 12 fichiers — un unique composant centralise structure et style, garantit que toute correction se propage partout, et permet de typer ses props (ex. `price: number`) pour éviter les incohérences constatées. `@apply` laisserait chaque copie du HTML divergente évoluer indépendamment, ce qui est précisément la source du problème observé.
