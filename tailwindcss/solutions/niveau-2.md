# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```html
<div class="bg-white dark:bg-gray-900 rounded-lg shadow p-6">
  <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100">Titre</h2>
  <p class="text-gray-600 dark:text-gray-400">Description de la carte.</p>
</div>
```

## Exercice 2

```css
.btn-primary {
  @apply bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700;
}
```

```html
<button class="btn-primary">Valider</button>
```

Dans un projet React, un composant `<Button>` serait préférable à `@apply` : il encapsule aussi le comportement (props, gestion des événements, variantes) et pas seulement le style, et reste l'unité de réutilisation naturelle du framework — `@apply` recrée un fichier CSS parallèle à maintenir, ce que Tailwind cherche justement à éviter.

## Exercice 3

```html
<label class="flex items-center gap-2">
  <input type="checkbox" class="peer" />
  <span class="peer-checked:text-green-600 peer-checked:underline">Actif</span>
</label>
```

## Exercice 4

```css
@theme {
  --color-brand: oklch(0.6 0.2 250);
}
```

```html
<button class="bg-brand text-white px-4 py-2 rounded-lg">Action</button>
```

## Exercice 5

```html
<input
  type="text"
  class="border rounded px-3 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none disabled:opacity-50 disabled:cursor-not-allowed"
  disabled
/>
```

L'état `focus:` est indispensable pour l'accessibilité clavier : un utilisateur qui navigue au `Tab` (sans souris) ne déclenche jamais `hover:`, donc sans indicateur `focus:` visible, il perd totalement le repère visuel de l'élément actuellement sélectionné.
