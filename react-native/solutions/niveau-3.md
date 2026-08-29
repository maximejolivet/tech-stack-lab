# Solutions — Niveau 3 (Avancé)

## Exercice 1

```tsx
const Row = React.memo(function Row({ item }: { item: Item }) {
  return (
    <View style={styles.row}>
      <Text>{item.title}</Text>
      <Text>{item.subtitle}</Text>
    </View>
  );
});

function ItemList({ items }: { items: Item[] }) {
  const renderItem = useCallback(({ item }: { item: Item }) => <Row item={item} />, []);
  return <FlatList data={items} keyExtractor={i => i.id} renderItem={renderItem} />;
}
```

`React.memo` sur `Row` évite de re-render une ligne si ses props (`item`) n'ont pas changé par référence. Mais si `renderItem` est recréé à chaque render du parent (nouvelle fonction inline à chaque fois), sa référence change constamment, ce qui invalide implicitement l'optimisation en cascade côté `FlatList` (qui compare aussi ses props). `useCallback` stabilise la référence de `renderItem` lui-même : les deux optimisations sont complémentaires, l'une protège le contenu de chaque ligne, l'autre protège la fonction qui les produit.

## Exercice 2

L'ancien pont fonctionne par sérialisation : chaque appel entre JS et natif (et inversement) est converti en JSON, mis en file d'attente, puis traité de façon **asynchrone** par batch — un aller-retour a un coût fixe de sérialisation/désérialisation, même pour une valeur simple. La nouvelle architecture (JSI) expose directement des références d'objets natifs au moteur JS, permettant des appels **synchrones** sans passer par une file JSON. Une interaction comme faire suivre un élément au doigt lors d'un `PanResponder`/geste bénéficie le plus de ce changement : chaque frame nécessite de lire une position native et de mettre à jour un style quasi instantanément, et la latence du pont JSON (même de quelques millisecondes) suffisait à créer un décalage visible entre le doigt et l'élément déplacé.

## Exercice 3

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withTiming } from 'react-native-reanimated';

function FadeInView({ children }: { children: React.ReactNode }) {
  const opacity = useSharedValue(0);

  useEffect(() => {
    opacity.value = withTiming(1, { duration: 500 });
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({ opacity: opacity.value }));

  return <Animated.View style={animatedStyle}>{children}</Animated.View>;
}
```

`Animated` (API de base) calcule les valeurs d'animation sur le thread JS puis les envoie au thread UI natif à chaque frame — si le thread JS est occupé (calcul lourd, re-render coûteux), l'animation peut saccader. `Reanimated` exécute la logique d'animation directement sur le thread UI natif (via des "worklets" compilés), rendant l'animation fluide même si le thread JS est momentanément bloqué.

## Exercice 4

`eas build --platform ios` envoie le code du projet à un service cloud managé par Expo, qui provisionne une machine macOS distante (nécessaire car Xcode ne tourne que sur macOS), y installe les dépendances natives (CocoaPods), compile l'app avec les certificats/profils de provisioning configurés (gérés automatiquement ou fournis par le développeur), puis produit un binaire `.ipa` téléchargeable ou directement soumissible à l'App Store. Cette approche cloud permet à un développeur sur Windows/Linux de produire un build iOS installable sans jamais posséder physiquement de matériel Apple.
