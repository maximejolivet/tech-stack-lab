# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```tsx
type User = { id: number; name: string };

function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);

    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then((data: User) => setUser(data))
      .finally(() => setLoading(false));

    return () => controller.abort();
  }, [userId]); // refetch si userId change

  if (loading) return <p>Chargement...</p>;
  return <p>{user?.name}</p>;
}
```

## Exercice 2

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? (JSON.parse(stored) as T) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

## Exercice 3

```tsx
function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const emailValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  const canSubmit = emailValid && password.length > 0;

  return (
    <form>
      <input value={email} onChange={e => setEmail(e.target.value)} placeholder="Email" />
      {!emailValid && email.length > 0 && <p>Email invalide</p>}
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
        placeholder="Mot de passe"
      />
      <button disabled={!canSubmit}>Se connecter</button>
    </form>
  );
}
```

## Exercice 4

```tsx
type Theme = "light" | "dark";
const ThemeContext = createContext<Theme>("light");

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme] = useState<Theme>("dark");
  return <ThemeContext.Provider value={theme}>{children}</ThemeContext.Provider>;
}

function useTheme() {
  return useContext(ThemeContext);
}

// Usage dans deux enfants distincts, sans prop drilling :
function Toolbar() { const theme = useTheme(); return <div className={theme}>Toolbar</div>; }
function Sidebar() { const theme = useTheme(); return <div className={theme}>Sidebar</div>; }
```

## Exercice 5

```tsx
function toFahrenheit(c: number) { return (c * 9) / 5 + 32; }
function toCelsius(f: number) { return ((f - 32) * 5) / 9; }

function TemperatureCalculator() {
  const [celsius, setCelsius] = useState(0);

  return (
    <>
      <label>
        °C
        <input
          type="number"
          value={celsius}
          onChange={e => setCelsius(Number(e.target.value))}
        />
      </label>
      <label>
        °F
        <input
          type="number"
          value={toFahrenheit(celsius)}
          onChange={e => setCelsius(toCelsius(Number(e.target.value)))}
        />
      </label>
    </>
  );
}
```

L'état (`celsius`) est remonté dans le parent commun `TemperatureCalculator` ; les deux inputs en dérivent leur valeur, ce qui garantit qu'ils restent synchronisés.
