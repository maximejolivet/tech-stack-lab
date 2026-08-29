# Exercices React Native — Niveau 2 (Intermédiaire)

## Exercice 1 — AsyncStorage

Crée un hook `useTheme()` qui lit un thème (`'light' | 'dark'`) depuis `AsyncStorage` au montage, expose une fonction `setTheme` qui met à jour à la fois le state local et `AsyncStorage`. Vérifie que le thème persiste après un rechargement de l'app.

## Exercice 2 — Navigation avec paramètres typés

Configure un Stack Navigator avec deux écrans `UserList` et `UserDetail`. Type les paramètres de route (`RootStackParamList`) pour que `UserDetail` reçoive un `userId: string` obligatoire. Navigue depuis `UserList` en passant l'id d'un utilisateur cliqué.

## Exercice 3 — Formulaire avec clavier

Crée un écran `AddNoteScreen` avec un `TextInput` (titre) et un `TextInput` multiline (contenu), enveloppés dans un `KeyboardAvoidingView`. Vérifie sur simulateur/émulateur que le clavier ne recouvre pas les champs actifs.

## Exercice 4 — Code spécifique par plateforme

Crée un composant `Header` dont le `paddingTop` diffère selon `Platform.OS` (44 sur iOS pour la status bar, 24 sur Android). Explique en une phrase pourquoi cette différence existe.

## Exercice 5 — Animation simple

Utilise `Animated.Value` et `Animated.timing` pour faire apparaître (fade-in, opacité de 0 à 1 sur 500ms) un composant au montage via `useEffect`.
