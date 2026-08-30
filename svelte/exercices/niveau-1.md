# Svelte — Exercices niveau 1 (Bases)

## Exercice 1 — Salutation réactive

Crée un composant `Greeting.svelte` avec une variable `$state` `name` initialisée à `'Max'`, un `<input bind:value={name}>`, et un affichage `Bonjour {name} !` qui se met à jour en temps réel.

## Exercice 2 — Compteur

Crée un composant `Counter.svelte` avec :
- une variable `$state` `count` initialisée à 0 ;
- un bouton "+1" qui incrémente `count` via `onclick` ;
- un bouton "Reset" qui remet `count` à 0 ;
- l'affichage du compteur dans le markup.

## Exercice 3 — Liste avec clé

À partir d'une variable `$state` tableau de tâches (`{ id, label, done }`), affiche la liste avec `{#each}` (clé `(task.id)` obligatoire), chaque tâche affichant `✅` ou `⏳` selon `done`.

## Exercice 4 — Rendu conditionnel

À partir de la même liste de tâches, affiche "Aucune tâche" avec `{#if}/{:else}` quand le tableau est vide, la liste sinon.

## Exercice 5 — Props avec valeur par défaut

Crée un composant `Alert.svelte` qui reçoit une prop `message` (obligatoire) et une prop `type` (`'info'` par défaut) via `$props()`, affiche le message dans une `<div>`, et appelle une prop-fonction `onClose` passée par le parent quand on clique sur un bouton "×".
