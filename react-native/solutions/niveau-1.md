# Solutions — Niveau 1 (Bases)

## Exercice 1

```tsx
function Profile() {
  return (
    <View style={styles.container}>
      <Image source={{ uri: 'https://example.com/avatar.png' }} style={{ width: 80, height: 80 }} />
      <Text>Max</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 16, alignItems: 'center' },
});
```

## Exercice 2

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <View>
      <Pressable onPress={() => setCount(c => c - 1)} style={({ pressed }) => ({ opacity: pressed ? 0.5 : 1 })}>
        <Text>-1</Text>
      </Pressable>
      <Text>{count}</Text>
      <Pressable onPress={() => setCount(c => c + 1)} style={({ pressed }) => ({ opacity: pressed ? 0.5 : 1 })}>
        <Text>+1</Text>
      </Pressable>
    </View>
  );
}
```

## Exercice 3

```tsx
function Row() {
  return (
    <View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>
      <View style={{ width: 50, height: 50, backgroundColor: 'red' }} />
      <View style={{ width: 50, height: 50, backgroundColor: 'green' }} />
      <View style={{ width: 50, height: 50, backgroundColor: 'blue' }} />
    </View>
  );
}
// flexDirection doit être précisé car la valeur par défaut d'une View est 'column'
// (empilement vertical), contrairement au web où un flex container part de 'row'.
```

## Exercice 4

```tsx
const fruits = [
  { id: '1', name: 'Pomme' },
  { id: '2', name: 'Banane' },
  { id: '3', name: 'Cerise' },
];

function FruitList() {
  return (
    <FlatList
      data={fruits}
      keyExtractor={item => item.id}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
}
```

## Exercice 5

```tsx
// ❌ Erreur : React Native lève une exception, le texte ne peut pas flotter
// directement dans une <View> — tout texte doit être enveloppé dans un <Text>.
function Broken() {
  return <View>Bonjour le monde</View>;
}

// ✅ Corrigé
function Fixed() {
  return <View><Text>Bonjour le monde</Text></View>;
}
```
