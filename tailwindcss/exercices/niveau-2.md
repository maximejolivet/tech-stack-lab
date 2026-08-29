# Exercices Tailwind CSS — Niveau 2 (Intermédiaire)

## Exercice 1 — Dark mode

Reprends la carte de l'exercice 1 du niveau 1 et ajoute le support du dark mode (`dark:`) : fond sombre, texte clair, en gardant une bonne lisibilité.

## Exercice 2 — @apply raisonné

Un bouton `.btn-primary` (`bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700`) est répété à l'identique à 8 endroits d'un projet **non** basé sur un framework à composants (HTML pur). Utilise `@apply` pour factoriser ce style, puis explique en une phrase pourquoi cette solution serait moins appropriée dans un projet React.

## Exercice 3 — group et peer

Crée une checkbox cachée (`peer`) suivie d'un label dont le texte devient vert et souligné (`peer-checked:text-green-600 peer-checked:underline`) quand la case est cochée, sans JavaScript.

## Exercice 4 — Thème personnalisé

Déclare une couleur de marque personnalisée `brand` dans le thème (via `@theme`), puis utilise-la dans une classe `bg-brand` sur un bouton.

## Exercice 5 — Formulaire accessible

Crée un champ de formulaire (`<input>`) avec un état `focus:` visible (`focus:ring-2 focus:ring-blue-400 focus:outline-none`) et un état `disabled:` grisé (`disabled:opacity-50 disabled:cursor-not-allowed`). Explique en une phrase pourquoi l'état `focus:` est important même si l'input a déjà un état `hover:`.
