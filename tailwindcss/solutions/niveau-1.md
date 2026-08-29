# Solutions — Niveau 1 (Bases)

## Exercice 1

```html
<div class="bg-white rounded-lg shadow p-6">
  <h2 class="text-xl font-bold">Titre</h2>
  <p class="text-gray-600">Description de la carte.</p>
</div>
```

## Exercice 2

```html
<button class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg">
  Valider
</button>
```

## Exercice 3

```html
<nav class="flex items-center justify-between gap-6 p-4">
  <span class="font-bold">Logo</span>
  <div class="flex gap-6">
    <a href="#">Accueil</a>
    <a href="#">Services</a>
    <a href="#">Contact</a>
  </div>
</nav>
```

## Exercice 4

```html
<div class="text-sm md:text-base lg:text-lg">
  Texte qui grossit progressivement
</div>
```

Tailwind est mobile-first : chaque préfixe (`md:`, `lg:`) s'applique **à partir de** ce breakpoint et vers le haut, donc l'ordre d'écriture n'a pas d'importance pour le résultat (le navigateur applique la dernière règle qui matche parmi celles dont le breakpoint est atteint) — mais omettre `md:text-base` ferait sauter directement de `text-sm` à `text-lg` à partir de `lg:`, sans palier intermédiaire.

## Exercice 5

```html
<div class="grid grid-cols-3 gap-4">
  <div class="col-span-2 bg-gray-100 rounded p-4">Carte 1 (large)</div>
  <div class="bg-gray-100 rounded p-4">Carte 2</div>
  <div class="bg-gray-100 rounded p-4">Carte 3</div>
  <div class="bg-gray-100 rounded p-4">Carte 4</div>
  <div class="bg-gray-100 rounded p-4">Carte 5</div>
  <div class="bg-gray-100 rounded p-4">Carte 6</div>
</div>
```
