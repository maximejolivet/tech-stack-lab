# Exercices React — Niveau 1 (Bases)

## Exercice 1 — Composant de salutation

Crée un composant `Greeting` qui reçoit une prop `name: string` et affiche "Bonjour, {name} !". Type les props avec TypeScript.

## Exercice 2 — Compteur

Crée un composant `Counter` avec un `useState` initialisé à 0, deux boutons "+1" et "-1", et un bouton "Reset" qui remet à 0. Utilise la forme fonctionnelle du setter.

## Exercice 3 — Liste avec clé

Étant donné un tableau `const fruits = [{ id: 1, name: "Pomme" }, { id: 2, name: "Banane" }, { id: 3, name: "Cerise" }]`, affiche-le sous forme de `<ul>` avec une `key` correcte (explique en commentaire pourquoi tu as choisi cette clé plutôt que l'index).

## Exercice 4 — Rendu conditionnel

Crée un composant `StatusBadge` qui reçoit une prop `isOnline: boolean` et affiche "🟢 En ligne" ou "🔴 Hors ligne" selon la valeur.

## Exercice 5 — Formulaire non contrôlé vs contrôlé

Explique en une phrase la différence entre un input contrôlé et non contrôlé, puis transforme cet input non contrôlé en input contrôlé :

```tsx
function SearchBox() {
  return <input type="text" placeholder="Rechercher..." />;
}
```
