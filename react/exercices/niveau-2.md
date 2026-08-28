# Exercices React — Niveau 2 (Intermédiaire)

## Exercice 1 — Fetch avec useEffect

Crée un composant `UserProfile` qui reçoit une prop `userId: number`, fait un fetch vers `https://jsonplaceholder.typicode.com/users/{userId}` dans un `useEffect`, affiche un état de chargement, puis le nom de l'utilisateur. Gère le cas où `userId` change (le composant doit refetch). Annule la requête en cours si le composant est démonté avant la fin du fetch.

## Exercice 2 — Hook personnalisé useLocalStorage

Écris un hook `useLocalStorage<T>(key: string, initialValue: T)` qui se comporte comme `useState` mais persiste automatiquement la valeur dans `localStorage`, et relit la valeur stockée au montage si elle existe.

## Exercice 3 — Formulaire contrôlé avec validation

Crée un formulaire de connexion (`email`, `password`) entièrement contrôlé. Affiche une erreur sous le champ email si le format n'est pas valide (regex simple), et désactive le bouton "Se connecter" tant que les deux champs ne sont pas valides.

## Exercice 4 — Context pour un thème

Crée un `ThemeContext` (`"light" | "dark"`), un `ThemeProvider`, et un hook `useTheme()` qui lit le contexte. Utilise-le dans deux composants enfants distincts sans passer la prop manuellement (pas de prop drilling).

## Exercice 5 — Lifting state up

Deux composants `TemperatureInput` (Celsius) et `TemperatureInput` (Fahrenheit) doivent rester synchronisés : si on tape dans l'un, l'autre se met à jour automatiquement. Implémente-le en remontant l'état dans leur parent commun.
