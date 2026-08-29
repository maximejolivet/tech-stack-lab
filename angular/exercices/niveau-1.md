# Exercices Angular — Niveau 1 (Bases)

## Exercice 1 — Composant de salutation

Crée un composant standalone `GreetingComponent` avec une propriété `name = 'Max'` et un template qui affiche `"Bonjour, {{ name }} !"` par interpolation.

## Exercice 2 — Compteur

Crée un composant `CounterComponent` avec une propriété `count = 0`, deux boutons "+1" et "-1" (event binding `(click)`), et un bouton "Reset" qui remet `count` à 0.

## Exercice 3 — Liste avec @for

Étant donné `items = [{ id: 1, name: 'Pomme' }, { id: 2, name: 'Banane' }, { id: 3, name: 'Cerise' }]`, affiche-les dans une `<ul>` avec `@for` et `track item.id`.

## Exercice 4 — Rendu conditionnel

Crée un composant `StatusBadgeComponent` avec une propriété `isOnline: boolean` et affiche "🟢 En ligne" ou "🔴 Hors ligne" avec `@if`/`@else`.

## Exercice 5 — Two-way binding

Importe `FormsModule` et crée un composant avec un `<input [(ngModel)]="username" />` qui affiche en temps réel `"Bonjour, {{ username }}"` juste en dessous.
