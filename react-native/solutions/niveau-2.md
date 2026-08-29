# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```tsx
function useTheme() {
  const [theme, setThemeState] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    AsyncStorage.getItem('theme').then(stored => {
      if (stored === 'light' || stored === 'dark') setThemeState(stored);
    });
  }, []);

  const setTheme = async (value: 'light' | 'dark') => {
    setThemeState(value);
    await AsyncStorage.setItem('theme', value);
  };

  return { theme, setTheme };
}
```

## Exercice 2

```tsx
type RootStackParamList = {
  UserList: undefined;
  UserDetail: { userId: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="UserList" component={UserListScreen} />
        <Stack.Screen name="UserDetail" component={UserDetailScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function UserListScreen({ navigation }: NativeStackScreenProps<RootStackParamList, 'UserList'>) {
  return (
    <Pressable onPress={() => navigation.navigate('UserDetail', { userId: '42' })}>
      <Text>Voir l'utilisateur 42</Text>
    </Pressable>
  );
}
```

## Exercice 3

```tsx
function AddNoteScreen() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  return (
    <KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : 'height'} style={{ flex: 1 }}>
      <TextInput value={title} onChangeText={setTitle} placeholder="Titre" />
      <TextInput value={content} onChangeText={setContent} placeholder="Contenu" multiline />
    </KeyboardAvoidingView>
  );
}
```

## Exercice 4

```tsx
const styles = StyleSheet.create({
  header: { paddingTop: Platform.OS === 'ios' ? 44 : 24 },
});
```

Cette différence existe car iOS et Android ont des hauteurs de status bar/encoche différentes selon les appareils et versions d'OS — React Native ne les uniformise pas automatiquement pour ce genre de valeur brute, il faut adapter manuellement (ou utiliser `useSafeAreaInsets` de `react-native-safe-area-context` pour une solution plus robuste).

## Exercice 5

```tsx
function FadeInView({ children }: { children: React.ReactNode }) {
  const opacity = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.timing(opacity, { toValue: 1, duration: 500, useNativeDriver: true }).start();
  }, []);

  return <Animated.View style={{ opacity }}>{children}</Animated.View>;
}
```
