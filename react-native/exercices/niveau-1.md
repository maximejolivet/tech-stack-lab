# Exercices React Native — Niveau 1 (Bases)

## Exercice 1 — Profil statique

Crée un composant `Profile` qui affiche une `Image` (URL de ton choix), un `Text` avec un nom, dans une `View` stylée avec `StyleSheet.create` (padding, alignItems centré).

## Exercice 2 — Compteur tactile

Crée un composant `Counter` avec un `useState` à 0, et deux `Pressable` ("+1" / "-1") qui modifient l'état. Change l'opacité du bouton pressé via la prop fonction de style de `Pressable`.

## Exercice 3 — Layout Flexbox

Crée une `View` avec `flexDirection: 'row'` contenant 3 `View` enfants colorées différemment, réparties avec `justifyContent: 'space-between'`. Explique en commentaire pourquoi `flexDirection` doit être précisé explicitement ici (valeur par défaut différente du web).

## Exercice 4 — FlatList simple

Étant donné `const fruits = [{ id: '1', name: 'Pomme' }, { id: '2', name: 'Banane' }, { id: '3', name: 'Cerise' }]`, affiche-les avec un `FlatList` (pas de `.map()`), en utilisant `keyExtractor`.

## Exercice 5 — Texte hors balise

Sans exécuter le code, prédis ce qui se passe avec ce composant, puis corrige-le :

```tsx
function Broken() {
  return <View>Bonjour le monde</View>;
}
```
