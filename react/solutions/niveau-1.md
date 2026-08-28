# Solutions — Niveau 1 (Bases)

## Exercice 1

```tsx
type GreetingProps = { name: string };

function Greeting({ name }: GreetingProps) {
  return <p>Bonjour, {name} !</p>;
}
```

## Exercice 2

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c - 1)}>-1</button>
      <span> {count} </span>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

## Exercice 3

```tsx
const fruits = [{ id: 1, name: "Pomme" }, { id: 2, name: "Banane" }, { id: 3, name: "Cerise" }];

function FruitList() {
  return (
    <ul>
      {fruits.map(fruit => (
        // key = fruit.id (identifiant stable des données), pas l'index :
        // si la liste est triée/filtrée plus tard, l'index changerait de sens
        // pour un même fruit et pourrait provoquer des bugs d'état mal associé.
        <li key={fruit.id}>{fruit.name}</li>
      ))}
    </ul>
  );
}
```

## Exercice 4

```tsx
function StatusBadge({ isOnline }: { isOnline: boolean }) {
  return <span>{isOnline ? "🟢 En ligne" : "🔴 Hors ligne"}</span>;
}
```

## Exercice 5

Un input **contrôlé** a sa valeur pilotée entièrement par le state React (`value` + `onChange`) ; un input **non contrôlé** garde son propre état DOM interne, lu ponctuellement via une `ref`.

```tsx
function SearchBox() {
  const [query, setQuery] = useState("");
  return (
    <input
      type="text"
      placeholder="Rechercher..."
      value={query}
      onChange={e => setQuery(e.target.value)}
    />
  );
}
```
