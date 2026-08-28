# Vue.js — Exercices niveau 1 (Bases)

## Exercice 1 — Compteur

Crée un composant `Counter.vue` avec :
- un `ref` `count` initialisé à 0 ;
- un bouton "+1" qui incrémente `count` ;
- un bouton "Reset" qui remet `count` à 0 ;
- l'affichage du compteur dans le template.

## Exercice 2 — Champ de saisie contrôlé

Crée un composant avec un `<input>` lié à un `ref` `text` via `v-model`, et affiche en dessous "Vous avez tapé : {{ text }}" ainsi que le nombre de caractères saisis.

## Exercice 3 — Liste conditionnelle

À partir d'un `ref` tableau de tâches (`{ id, label, done }`), affiche :
- la liste complète avec `v-for` (et `:key` correct) ;
- un message "Aucune tâche" si le tableau est vide, en utilisant `v-if`/`v-else`.

## Exercice 4 — Props et emit

Crée un composant `Alert.vue` qui reçoit une prop `message` (string, obligatoire) et un prop `type` (string, `'info'` par défaut), affiche le message dans un `<div>`, et émet un événement `close` quand on clique sur un bouton "×". Utilise cet événement depuis le composant parent pour retirer l'alerte de l'affichage.
