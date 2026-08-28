# Solutions — Niveau 3 (Avancé)

## Exercice 1

```tsx
function ProductList({ products, searchTerm }: { products: Product[]; searchTerm: string }) {
  const filtered = useMemo(
    () => products.filter(p => p.name.includes(searchTerm)),
    [products, searchTerm] // ne recalcule que si products ou searchTerm changent
  );
  return <ul>{filtered.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

Le `useMemo` doit être placé sur le calcul du filtrage des 10 000 produits, avec `[products, searchTerm]` en dépendances — le champ "commentaire", non présent dans ces dépendances, ne déclenchera plus de recalcul.

**Pourquoi ne pas généraliser `useMemo` partout** : chaque `useMemo` a un coût (stockage de la valeur précédente + comparaison des dépendances à chaque render). Sur un calcul trivial (ex: `a + b`), ce coût dépasse le gain, et ça ajoute de la complexité de lecture pour rien. À réserver aux calculs réellement coûteux (mesurés, pas supposés).

## Exercice 2

```tsx
const Home = React.lazy(() => import("./pages/Home"));
const Settings = React.lazy(() => import("./pages/Settings"));
const Admin = React.lazy(() => import("./pages/Admin"));

function App() {
  return (
    <Suspense fallback={<p>Chargement...</p>}>
      <RouterProvider router={router} /> {/* router référence Home/Settings/Admin */}
    </Suspense>
  );
}
```

Gain attendu : le JS de `Settings` et `Admin` n'est téléchargé qu'au moment où l'utilisateur navigue vers ces pages, réduisant la taille du bundle initial et donc le temps de chargement de la première page.

## Exercice 3

```tsx
const TabsContext = createContext<{ active: string; setActive: (id: string) => void } | null>(null);

function Tabs({ children, defaultTab }: { children: React.ReactNode; defaultTab: string }) {
  const [active, setActive] = useState(defaultTab);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}

function useTabsContext() {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error("Tabs.* doit être utilisé dans <Tabs>");
  return ctx;
}

Tabs.List = function TabsList({ children }: { children: React.ReactNode }) {
  return <div role="tablist">{children}</div>;
};

Tabs.Tab = function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const { active, setActive } = useTabsContext();
  return (
    <button aria-selected={active === id} onClick={() => setActive(id)}>
      {children}
    </button>
  );
};

Tabs.Panel = function Panel({ id, children }: { id: string; children: React.ReactNode }) {
  const { active } = useTabsContext();
  return active === id ? <div>{children}</div> : null;
};
```

## Exercice 4

```tsx
import { render, screen, fireEvent } from "@testing-library/react";

test("incrémente le compteur au clic", () => {
  render(<Counter />);
  expect(screen.getByText("0")).toBeInTheDocument();

  fireEvent.click(screen.getByText("+1"));

  expect(screen.getByText("1")).toBeInTheDocument();
});
```

Le test interagit uniquement via `screen` (ce que voit l'utilisateur) et `fireEvent` (ce qu'il fait) — aucun accès à `useState` ou aux variables internes du composant.

## Exercice 5

Le re-render se produit car une **nouvelle référence** de `data` (ou d'une fonction passée en prop) est créée à chaque render du parent, même si son contenu est identique — `React.memo` compare les props par référence (`===`), pas par valeur profonde.

```tsx
function Parent() {
  const [search, setSearch] = useState("");
  const data = useMemo(() => computeChartData(rawData), [rawData]); // référence stable
  const handleSelect = useCallback((id: string) => { /* ... */ }, []); // référence stable

  return (
    <>
      <input value={search} onChange={e => setSearch(e.target.value)} />
      <ExpensiveChart data={data} onSelect={handleSelect} />
    </>
  );
}

const ExpensiveChart = React.memo(function ExpensiveChart({ data, onSelect }: ChartProps) {
  // ne re-render que si data ou onSelect changent réellement de référence
  return /* ... */;
});
```
