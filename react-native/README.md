# React Native

## 1. Introduction

React Native permet de construire des applications mobiles natives (iOS/Android) en réutilisant le modèle mental et la syntaxe de React ([`../react/`](../react/)) — composants, hooks, JSX — mais en rendant de vrais composants natifs plutôt que des éléments DOM. Ce dossier suppose React déjà maîtrisé et se concentre sur ce qui **change** côté mobile.

**À quoi sert-il ?**
- Construire une application mobile pour iOS et Android à partir d'une base de code JavaScript/TypeScript très largement partagée.
- Réutiliser les compétences et parfois une partie du code d'une équipe déjà à l'aise avec React web.

**Où se situe-t-il dans une architecture mobile ?** React Native exécute du JavaScript (moteur Hermes) qui communique avec le monde natif (iOS/Android) via un pont (bridge historique) ou, avec la nouvelle architecture, JSI (JavaScript Interface) et des Turbo Modules — le JS ne "dessine" jamais directement l'écran : il pilote de vrais composants UI natifs (`UIView` sur iOS, `View` Android).

**Avantages**
- Un seul codebase pour iOS et Android (avec possibilité de code spécifique par plateforme si besoin), gain de temps considérable face à deux apps natives séparées.
- Hot reload / Fast Refresh : itération très rapide, proche du confort de développement web.
- Écosystème React directement transférable (hooks, state management, patterns de composition).

**Limites**
- Performance légèrement en retrait face à du natif pur (Swift/Kotlin) sur des animations très complexes ou du calcul intensif, bien que la nouvelle architecture (Fabric, JSI) réduise significativement cet écart.
- Accès aux API natives récentes/spécifiques parfois en retard, nécessite occasionnellement d'écrire un module natif custom.
- Deux plateformes à tester malgré tout : certains comportements (clavier, permissions, notifications) divergent réellement entre iOS et Android et ne peuvent pas être ignorés.

## 2. Prérequis

- React solide ([`../react/`](../react/)) : hooks, composants, props, gestion d'état — tout est directement transférable.
- Node.js et npm/yarn installés ; un compte Expo (gratuit) simplifie considérablement le démarrage sans configuration native lourde.
- Un simulateur iOS (macOS + Xcode) ou émulateur Android (Android Studio) pour tester localement, ou l'app Expo Go sur un téléphone physique pour démarrer sans rien installer de lourd.

## 3. Rappel des bases 🟢

### 01 - Créer un projet (Expo)

**Explication** — [Expo](https://expo.dev) est la façon standard moderne de démarrer : gère le build natif, l'accès aux API device courantes, et permet de tester instantanément sur un téléphone via l'app Expo Go (scan d'un QR code), sans installer Xcode/Android Studio pour commencer.

```bash
npx create-expo-app mon-app
cd mon-app && npx expo start
```

**Bonne pratique** : démarrer avec Expo par défaut ; ne passer en "bare workflow" (éjection, accès natif complet) que si un besoin précis l'exige (module natif custom non supporté par Expo).

### 02 - Composants natifs de base

**Explication** — Pas de HTML : React Native fournit ses propres composants qui se compilent en éléments natifs. `View` remplace `div`, `Text` remplace tout texte (obligatoire — impossible d'afficher du texte brut hors d'un `<Text>`), `Image` remplace `img`, `ScrollView`/`FlatList` remplacent le scroll natif du navigateur (inexistant par défaut sur mobile).

```tsx
import { View, Text, Image } from 'react-native';

function Profile() {
  return (
    <View>
      <Image source={{ uri: 'https://example.com/avatar.png' }} style={{ width: 80, height: 80 }} />
      <Text>Max</Text>
    </View>
  );
}
```

**Erreur fréquente** : écrire `<div>` ou du texte brut hors d'un `<Text>` (`<View>Bonjour</View>`) — React Native lève une erreur explicite, contrairement au web où le texte flotte librement dans n'importe quel conteneur.

### 03 - StyleSheet et Flexbox

**Explication** — Pas de CSS : les styles sont des objets JavaScript, définis via `StyleSheet.create` (optimisé, référence stable) plutôt qu'inline à chaque render. Le layout est **Flexbox par défaut** sur tous les conteneurs (contrairement au web où `display: block` est la valeur par défaut).

```tsx
import { StyleSheet, View, Text } from 'react-native';

function Card() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Titre</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flexDirection: 'row', padding: 16, backgroundColor: '#fff' },
  title: { fontSize: 18, fontWeight: 'bold' },
});
```

**Cas d'usage** : `flexDirection: 'column'` est la valeur par défaut de `View` (contrairement au web où `flex-direction: row` est le défaut d'un flex container) — les éléments s'empilent verticalement sans configuration.

### 04 - Gestion des événements tactiles

**Explication** — Pas d'`onClick` (pas de souris) : `Pressable` (recommandé, remplace les anciens `TouchableOpacity`/`TouchableHighlight`) gère les interactions tactiles avec des états (`pressed`) et retours visuels configurables.

```tsx
import { Pressable, Text } from 'react-native';

function Button({ onPress, label }: { onPress: () => void; label: string }) {
  return (
    <Pressable onPress={onPress} style={({ pressed }) => ({ opacity: pressed ? 0.5 : 1 })}>
      <Text>{label}</Text>
    </Pressable>
  );
}
```

**Bonne pratique** : préférer `Pressable` aux anciens composants `TouchableXxx` — plus flexible (accès à l'état `pressed` pour du style conditionnel) et c'est l'API recommandée actuelle.

### 05 - Listes performantes (FlatList)

**Explication** — Contrairement au web où `.map()` sur un tableau reste raisonnable, une longue liste mobile doit utiliser `FlatList` (ou `FlashList`, plus performant) : elle ne rend que les éléments visibles à l'écran (virtualisation), recyclant les vues au scroll.

```tsx
import { FlatList, Text } from 'react-native';

function UserList({ users }: { users: { id: string; name: string }[] }) {
  return (
    <FlatList
      data={users}
      keyExtractor={item => item.id}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
}
```

**Erreur fréquente** : faire un `.map()` direct dans un `ScrollView` pour une liste potentiellement longue — toutes les lignes sont montées en mémoire d'un coup, dégrade sévèrement les performances au-delà de quelques dizaines d'éléments.

### 06 - Navigation (React Navigation)

**Explication** — Comme React Router côté web, React Native n'a pas de routage natif : React Navigation est la librairie standard, avec des "navigators" (Stack, Tab, Drawer) qui reproduisent les patterns natifs de navigation mobile (pile d'écrans, onglets).

```tsx
const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

**Bonne pratique** : typer les paramètres de route avec TypeScript (`type RootStackParamList = { Home: undefined; Profile: { userId: string } }`) pour éviter des erreurs de navigation silencieuses.

## 4. Concepts intermédiaires 🟡

- **State management** : identique à React web ([`../react/`](../react/)) — `useState`, `useEffect`, Context, ou libs tierces (Zustand) fonctionnent à l'identique, aucune notion supplémentaire spécifique au mobile.
- **Stockage local persistant** : pas de `localStorage` (API web absente sur mobile) — `AsyncStorage` (ou `expo-secure-store` pour des données sensibles comme des tokens) est l'équivalent asynchrone standard.

```tsx
import AsyncStorage from '@react-native-async-storage/async-storage';

await AsyncStorage.setItem('theme', 'dark');
const theme = await AsyncStorage.getItem('theme');  // toujours asynchrone, contrairement à localStorage
```

- **Gestion du clavier** : le clavier virtuel peut recouvrir des champs de saisie — `KeyboardAvoidingView` ajuste automatiquement la mise en page pour garder le champ actif visible, un problème qui n'existe pas sur le web.
- **Code spécifique par plateforme** : `Platform.OS === 'ios' | 'android'`, ou fichiers suffixés `Component.ios.tsx` / `Component.android.tsx` résolus automatiquement au build selon la plateforme cible.

```tsx
import { Platform } from 'react-native';

const styles = StyleSheet.create({
  header: { paddingTop: Platform.OS === 'ios' ? 44 : 24 },  // hauteur de status bar différente
});
```

- **Permissions et API device** : accès caméra, géolocalisation, notifications via des modules Expo (`expo-camera`, `expo-location`) qui gèrent la demande de permission utilisateur de façon unifiée entre iOS et Android, malgré des dialogues natifs différents.
- **Animations (Animated API / Reanimated)** : `Animated` (intégré) pour des animations simples pilotées par le thread JS ; `react-native-reanimated` (standard en production) exécute les animations sur le thread UI natif, évitant les saccades liées à la charge du thread JS.

## 5. Concepts avancés 🟠🔴

- **La nouvelle architecture (Fabric et Turbo Modules)** : remplace le pont JSON historique (sérialisation asynchrone entre JS et natif, source de latence) par JSI (JavaScript Interface), qui permet un appel direct et synchrone entre JS et code natif — impact direct sur la fluidité des animations et la réactivité des interactions complexes.
- **Le moteur Hermes** : moteur JavaScript optimisé pour mobile (développé par Meta), compile le bytecode en amont (au build) plutôt qu'au démarrage de l'app — démarrage plus rapide et empreinte mémoire réduite comparé à l'utilisation directe de JSC (JavaScriCore) ou V8.
- **Modules natifs custom** : quand une API native n'est pas couverte par Expo/React Native, il est possible d'écrire un module natif en Swift/Kotlin exposé au JS via Turbo Modules — pont vers l'écosystème natif complet quand c'est réellement nécessaire.
- **Performance des listes et re-renders** : `FlatList` accepte des props d'optimisation (`getItemLayout` pour éviter un calcul de mesure, `windowSize` pour ajuster combien d'éléments hors écran restent montés) ; comme en React web, `React.memo`/`useCallback` évitent les re-renders inutiles de `renderItem`.
- **Build et déploiement (EAS)** : Expo Application Services (EAS) gère le build cloud (pas besoin d'un Mac local pour builder iOS) et la soumission aux stores (App Store, Google Play) — remplace le processus manuel de build natif via Xcode/Android Studio.

```bash
eas build --platform ios       # build cloud, sans Mac local requis
eas submit --platform ios         # soumission à l'App Store
```

## 6. Commandes / syntaxe à connaître

```bash
npx create-expo-app mon-app        # créer un projet
npx expo start                      # serveur de dev, QR code pour Expo Go
npx expo start --android              # ouvrir directement sur émulateur Android
eas build --platform all                # build cloud iOS + Android
```

```tsx
<View style={styles.container}><Text>...</Text></View>
<Pressable onPress={() => {}}>...</Pressable>
<FlatList data={items} keyExtractor={i => i.id} renderItem={({ item }) => <Text>{item.name}</Text>} />
await AsyncStorage.setItem('key', value);
Platform.OS === 'ios'
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Application de notes avec persistance locale**

Construire une petite application React Native (Expo) qui doit :
- Un écran `NoteListScreen` affichant les notes via `FlatList` (titre + aperçu du contenu).
- Un écran `NoteFormScreen` (via React Navigation, Stack Navigator) pour créer/éditer une note, avec `KeyboardAvoidingView` pour garder le formulaire visible.
- Persister les notes avec `AsyncStorage` (lecture au montage, écriture à chaque modification).
- Un `Pressable` sur chaque note qui navigue vers l'écran d'édition avec l'id de la note en paramètre typé.
- Bonus : ajouter `Platform.OS` pour adapter légèrement le style du header entre iOS et Android.

Objectif : mobiliser composants natifs, FlatList, navigation, AsyncStorage et gestion du clavier dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux (composants natifs, StyleSheet, Flexbox)
- [ ] Savoir créer un projet Expo et le lancer sur Expo Go
- [ ] Maîtriser la syntaxe principale (Pressable, FlatList, navigation)
- [ ] Comprendre les concepts importants (AsyncStorage, code spécifique par plateforme, clavier)
- [ ] Savoir debugger (React Native DevTools, logs Expo)
- [ ] Connaître les bonnes pratiques (FlatList vs .map, Pressable vs Touchable*)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (nouvelle architecture JSI, Hermes, EAS build)

## 10. Ressources

- [Documentation officielle React Native](https://reactnative.dev) — référence des composants et API natives.
- [Documentation Expo](https://docs.expo.dev) — référence du workflow Expo et des modules disponibles.
- [React Navigation](https://reactnavigation.org) — documentation officielle du routeur.
- [roadmap.sh — React Native](https://roadmap.sh/react-native) — vue d'ensemble du parcours d'apprentissage.
