# React

## 1. Introduction

React est une bibliothèque JavaScript pour construire des interfaces utilisateur déclaratives à base de composants. Ce dossier couvre **React 18+ avec les hooks et les function components** (les class components, devenues legacy, ne sont pas détaillées). Le JS/TS sous-jacent est déjà couvert dans [`../javascript/`](../javascript/) et [`../typescript/`](../typescript/) — pas de rappel ici.

**À quoi sert-il ?**
- Construire des UI complexes en les découpant en composants réutilisables et composables.
- Synchroniser automatiquement l'UI avec un état (`state`) qui change dans le temps.

**Problèmes résolus** : avant React (ou Vue/Angular), manipuler le DOM manuellement à chaque changement d'état devenait ingérable à l'échelle d'une vraie application (bugs de synchronisation, code impératif difficile à maintenir). React propose un modèle **déclaratif** : on décrit à quoi l'UI doit ressembler pour un état donné, React se charge de la mettre à jour.

**Où se situe-t-il dans une architecture web ?** Côté client (SPA) ou avec du rendu serveur via des frameworks comme Next.js (non traité ici, hors scope de la roadmap actuelle — voir Ressources). React lui-même ne gère ni routing ni state global : ce sont des bibliothèques tierces (React Router, Zustand/Redux) qui complètent l'écosystème.

**Avantages** : écosystème immense, un seul modèle mental (composants + hooks) pour toute l'app, excellent outillage (DevTools), forte demande sur le marché.
**Limites** : beaucoup de décisions déléguées à des libs tierces (routing, state, formulaires — contrairement à Angular "batteries included"), JSX demande un temps d'adaptation, pièges classiques avec `useEffect` si mal compris.

## 2. Prérequis

- JavaScript solide : closures, `this`, destructuring, spread/rest, modules ES, async/await (voir [`../javascript/`](../javascript/)).
- Idéalement TypeScript pour les exemples typés de ce dossier (voir [`../typescript/`](../typescript/)), même si les bases restent lisibles en JS pur.
- Node.js et npm installés pour lancer un projet local.

## 3. Rappel des bases 🟢

### 01 - Créer un projet React

**Explication** — Le standard actuel est [Vite](https://vitejs.dev), rapide et sans configuration lourde (create-react-app est déprécié).

```bash
npm create vite@latest mon-app -- --template react-ts
cd mon-app && npm install && npm run dev
```

**Bonne pratique** : toujours démarrer un nouveau projet avec le template `react-ts` (TypeScript), même pour un petit projet — le coût d'apprentissage est rentabilisé dès que le projet grossit.

### 02 - JSX

**Explication** — JSX est une extension de syntaxe qui permet d'écrire du "HTML" dans du JS ; il est compilé en appels `React.createElement(...)`. Ce n'est **pas** du HTML : ce sont des expressions JavaScript.

```jsx
function Greeting({ name }) {
  const isMorning = new Date().getHours() < 12;
  return (
    <div className="greeting">
      <h1>{isMorning ? "Bonjour" : "Bonsoir"} {name}</h1>
    </div>
  );
}
```

**Différences clés avec HTML** : `className` au lieu de `class`, `htmlFor` au lieu de `for`, attributs en camelCase (`onClick`, `tabIndex`), un JSX doit retourner un seul élément racine (ou un Fragment `<>...</>`), `{}` insère n'importe quelle expression JS.

**Erreur fréquente** : essayer d'insérer une instruction (`if`, `for`) directement dans le JSX — seules des **expressions** sont autorisées (utiliser un opérateur ternaire ou extraire la logique avant le `return`).

**Bonne pratique** : garder le JSX simple ; extraire la logique complexe dans des variables ou fonctions avant le `return`.

### 03 - Function components & props

**Explication** — Un composant est une fonction qui reçoit un objet `props` en entrée et retourne du JSX. Les props sont **en lecture seule** (immuables côté enfant).

```tsx
type CardProps = { title: string; children: React.ReactNode };

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Usage : <Card title="Profil">Contenu ici</Card>
```

**Erreur fréquente** : muter directement une prop (`props.title = "x"`) — React ne détectera pas le changement et ça casse le flux de données unidirectionnel (parent → enfant).

**Bonne pratique** : typer systématiquement les props avec TypeScript ; utiliser `children` pour la composition plutôt que de multiplier les props de contenu.

### 04 - useState (state local)

**Explication** — `useState` déclare une variable d'état locale au composant. Chaque appel au setter déclenche un **re-render** du composant.

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>Compteur : {count}</button>;
}
```

**Erreur fréquente** — mettre à jour un état à partir de sa propre valeur sans utiliser la forme fonctionnelle, dans un contexte où plusieurs mises à jour peuvent s'enchaîner (ex: plusieurs `setCount(count + 1)` de suite ne donnent pas +2 mais souvent +1, car `count` est figé dans la closure du render courant) :

```tsx
// ❌ Risque de valeur obsolète
setCount(count + 1);
setCount(count + 1);

// ✅ Forme fonctionnelle, toujours basée sur la dernière valeur
setCount(c => c + 1);
setCount(c => c + 1);
```

**Bonne pratique** : utiliser la forme fonctionnelle du setter dès que la nouvelle valeur dépend de l'ancienne.

### 05 - Rendu conditionnel et listes

**Explication** — Rendu conditionnel via ternaire, `&&`, ou variable ; les listes se rendent avec `.map()`, chaque élément nécessitant une prop `key` **stable et unique**.

```tsx
{isLoggedIn ? <Dashboard /> : <LoginForm />}
{items.length > 0 && <ul>{items.map(item => <li key={item.id}>{item.label}</li>)}</ul>}
```

**Pourquoi la `key` est importante** : React s'en sert pour identifier quel élément du DOM correspond à quel élément du tableau entre deux renders, afin de réutiliser/déplacer les nœuds au lieu de tout recréer.

**Erreur fréquente** : utiliser l'**index du tableau** comme `key` quand la liste peut être réordonnée/filtrée — ça provoque des bugs d'état mal associé (ex: un input qui garde la valeur du mauvais élément après suppression).

**Bonne pratique** : toujours utiliser un identifiant stable et unique venant des données (`item.id`), jamais l'index sauf liste strictement statique et non réordonnable.

### 06 - Gestion des événements

**Explication** — Les handlers sont passés en camelCase sous forme de référence de fonction (pas d'appel direct).

```tsx
function Form() {
  function handleSubmit(e: React.FormEvent) {
    e.preventDefault(); // empêcher le rechargement de page
    console.log("soumis");
  }
  return <form onSubmit={handleSubmit}><button type="submit">Envoyer</button></form>;
}
```

**Erreur fréquente** : écrire `onClick={handleClick()}` au lieu de `onClick={handleClick}` — la première forme **appelle** la fonction immédiatement au render au lieu de la passer en référence.

**Bonne pratique** : utiliser une arrow function inline uniquement pour passer des arguments (`onClick={() => handleDelete(item.id)}`), sinon passer la référence directement.

### 07 - Composition (children)

**Explication** — `children` est une prop spéciale qui contient tout ce qui est passé entre les balises ouvrantes/fermantes d'un composant ; c'est le mécanisme de composition principal de React (préférer la composition à l'héritage, comme en [design patterns](../design-patterns/)).

```tsx
function Layout({ children }: { children: React.ReactNode }) {
  return <div className="layout"><Header /><main>{children}</main><Footer /></div>;
}
```

**Bonne pratique** : privilégier la composition de petits composants dédiés plutôt qu'un composant unique avec de nombreuses props conditionnelles.

## 4. Concepts intermédiaires 🟡

### 08 - useEffect

**Explication** — `useEffect` exécute un effet de bord (fetch, abonnement, manipulation DOM directe) après le rendu, et peut le "nettoyer" avant le prochain effet ou le démontage du composant. Le tableau de dépendances contrôle quand l'effet se relance.

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setUser);

  return () => controller.abort(); // cleanup
}, [userId]); // relance uniquement si userId change
```

**Erreurs fréquentes** :
- Oublier une dépendance dans le tableau → bug de "stale closure" (l'effet utilise une valeur périmée). Le linter `eslint-plugin-react-hooks` détecte ce cas, ne jamais le désactiver sans comprendre pourquoi.
- Oublier le tableau de dépendances entier → l'effet tourne à **chaque** render, souvent une boucle infinie si l'effet modifie un state qu'il lit.
- Utiliser `useEffect` pour dériver une valeur à partir des props/state (ex: calculer un total) — un calcul simple pendant le render suffit, pas besoin d'effet.

**Bonne pratique** : `useEffect` sert à synchroniser avec un système **externe** (réseau, DOM, timer, souscription) — pas à orchestrer la logique interne du composant.

### 09 - Hooks personnalisés

**Explication** — Une fonction dont le nom commence par `use` et qui peut appeler d'autres hooks, pour extraire et réutiliser de la logique stateful.

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(url).then(r => r.json()).then(setData).finally(() => setLoading(false));
  }, [url]);

  return { data, loading };
}
```

**Bonne pratique** : dès qu'une logique `useState` + `useEffect` se répète dans plusieurs composants, l'extraire en hook personnalisé.

### 10 - useContext

**Explication** — Permet de partager une valeur (state, thème, utilisateur connecté) à travers l'arbre de composants sans passer les props manuellement à chaque niveau ("prop drilling").

```tsx
const ThemeContext = createContext<"light" | "dark">("light");

function App() {
  return <ThemeContext.Provider value="dark"><Toolbar /></ThemeContext.Provider>;
}
function Toolbar() {
  const theme = useContext(ThemeContext); // accessible sans prop drilling
  return <div className={theme}>...</div>;
}
```

**Erreur fréquente** : utiliser le Context pour **tout** l'état de l'app (y compris des données qui changent souvent) — chaque changement de valeur re-render tous les consommateurs, ce qui peut nuire aux performances. Réservé aux valeurs globales peu fréquemment mises à jour (thème, utilisateur, langue) ; pour du state fréquent et complexe, voir une lib dédiée (Zustand, Redux).

### 11 - Lifting state up

**Explication** — Quand deux composants enfants doivent partager un état, on le remonte dans leur ancêtre commun le plus proche, qui le redescend en props.

```tsx
function Parent() {
  const [value, setValue] = useState("");
  return (
    <>
      <Input value={value} onChange={setValue} />
      <Preview value={value} />
    </>
  );
}
```

**Bonne pratique** : ne remonter l'état que jusqu'au strict ancêtre commun nécessaire — remonter systématiquement "trop haut" complexifie inutilement l'arbre de props.

### 12 - Formulaires contrôlés

**Explication** — Un input "contrôlé" a sa valeur pilotée par le state React (`value` + `onChange`), par opposition à un input "non contrôlé" (valeur lue via `useRef` au besoin).

```tsx
function LoginForm() {
  const [email, setEmail] = useState("");
  return <input value={email} onChange={e => setEmail(e.target.value)} />;
}
```

**Erreur fréquente** : mélanger contrôlé et non contrôlé (donner une `value` sans `onChange`) → React log un warning et l'input devient en lecture seule de fait.

**Bonne pratique** : pour des formulaires complexes avec validation, s'appuyer sur une lib dédiée (React Hook Form) plutôt que de tout gérer à la main avec `useState`.

### 13 - useRef

**Explication** — Contient une valeur mutable qui **survit** aux re-renders sans en déclencher (contrairement à `useState`). Deux usages principaux : référencer un élément DOM, ou stocker une valeur mutable "hors rendu" (ex: un ID de timer).

```tsx
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  useEffect(() => { inputRef.current?.focus(); }, []);
  return <input ref={inputRef} />;
}
```

**Erreur fréquente** : utiliser `useRef` pour du state qui doit déclencher un re-render à l'écran — dans ce cas, c'est `useState` qu'il faut, pas `useRef` (qui est "silencieux").

### 14 - React Router (bases)

**Explication** — Bibliothèque tierce standard pour le routing côté client (React ne fournit pas de routeur natif).

```tsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  { path: "/", element: <Home /> },
  { path: "/users/:id", element: <UserProfile /> },
]);

function App() { return <RouterProvider router={router} />; }
```

**Bonne pratique** : utiliser le `loader` de React Router (data router) pour charger les données d'une route avant son affichage plutôt qu'un `useEffect` dans le composant, quand c'est possible.

### 15 - Gestion d'état globale (aperçu)

**Explication** — Au-delà du Context (peu adapté à du state fréquent), des libs dédiées existent : Zustand (simple, API minimaliste, basée sur des hooks) ou Redux Toolkit (plus structuré, utile sur de très gros projets avec des flux de données complexes).

```tsx
// Exemple Zustand
const useStore = create<{ count: number; inc: () => void }>((set) => ({
  count: 0,
  inc: () => set(s => ({ count: s.count + 1 })),
}));
// Usage : const count = useStore(s => s.count);
```

**Bonne pratique** : ne pas introduire de lib de state global "par défaut" — commencer par le state local + lifting, et n'ajouter Zustand/Redux que quand le partage d'état devient réellement complexe/transversal.

## 5. Concepts avancés 🟠🔴

### 16 - Performance : useMemo, useCallback, React.memo

**Explication** — Trois outils de mémoïsation, à utiliser avec parcimonie : `useMemo` mémorise une **valeur** calculée coûteuse, `useCallback` mémorise une **référence de fonction** (utile si elle est passée à un enfant mémoïsé ou en dépendance d'effet), `React.memo` évite le re-render d'un composant si ses props n'ont pas changé.

```tsx
const sortedItems = useMemo(() => items.toSorted((a, b) => a.price - b.price), [items]);
const handleClick = useCallback(() => onSelect(id), [id, onSelect]);
const ExpensiveRow = React.memo(function Row({ item }: { item: Item }) { /* ... */ });
```

**Erreur fréquente / piège** : mémoïser "par réflexe" partout — chaque `useMemo`/`useCallback` a un coût (comparaison des dépendances à chaque render) qui peut dépasser le gain sur des calculs triviaux. **Ne pas optimiser prématurément** : profiler d'abord (React DevTools Profiler) pour identifier un vrai goulot avant d'ajouter de la mémoïsation.

**Bonne pratique** : mémoïser surtout quand (1) le calcul est réellement coûteux, ou (2) la référence stable est nécessaire pour éviter un re-render en cascade dans un enfant mémoïsé.

### 17 - Reconciliation et Virtual DOM

**Explication** — À chaque changement d'état, React construit un nouvel arbre d'éléments (Virtual DOM), le compare (diff) à l'arbre précédent, et applique au DOM réel uniquement les changements minimaux nécessaires. C'est ce qui rend React performant sans manipulation manuelle du DOM. La `key` (voir section 05) est l'indice principal utilisé par l'algorithme pour savoir si un élément d'une liste a été déplacé, ajouté ou supprimé plutôt que recréé.

**Bonne pratique** : comprendre que "re-render" ≠ "re-peindre le DOM" — un composant peut re-render (ré-exécuter sa fonction) sans qu'aucune modification réelle du DOM n'ait lieu, si le diff ne détecte aucun changement.

### 18 - Server Components (aperçu conceptuel)

**Explication** — Les React Server Components (RSC) s'exécutent uniquement côté serveur, n'envoient aucun JS au client pour leur propre rendu, et peuvent accéder directement à des ressources serveur (base de données, fichiers). Ils se combinent avec des Client Components (`"use client"`) pour l'interactivité. Ce modèle est principalement utilisé via des frameworks comme Next.js (non couvert dans ce dossier) — à connaître conceptuellement même sans l'utiliser au quotidien avec du React "pur" (Vite SPA).

### 19 - Suspense et lazy loading

**Explication** — `Suspense` permet d'afficher un fallback (spinner) pendant qu'un composant ou des données sont en cours de chargement. `React.lazy` charge un composant à la demande (code-splitting).

```tsx
const Settings = React.lazy(() => import("./Settings"));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Settings />
    </Suspense>
  );
}
```

**Cas d'usage** : découper le bundle par route pour réduire le temps de chargement initial (lié aux notions de performance front, voir [`../accessibility-performance/`](../accessibility-performance/)).

### 20 - Patterns avancés

**Compound components** — plusieurs composants qui partagent un état implicite via Context pour offrir une API flexible et lisible (ex: `<Tabs><Tabs.List><Tabs.Panel>`).

**Render props / composants headless** — passer une fonction en prop (ou `children` en fonction) pour déléguer le rendu tout en gardant la logique dans le composant parent. Moins courant depuis les hooks personnalisés, mais encore utile pour des libs UI headless.

### 21 - Tests de composants

**Explication** — La philosophie recommandée (React Testing Library) est de **tester le comportement du point de vue de l'utilisateur**, pas les détails d'implémentation internes.

```tsx
import { render, screen, fireEvent } from "@testing-library/react";

test("incrémente le compteur au clic", () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole("button"));
  expect(screen.getByText(/Compteur : 1/)).toBeInTheDocument();
});
```

**Erreur fréquente** : tester des détails internes (state, noms de méthodes) plutôt que ce que voit/fait l'utilisateur — un tel test casse au moindre refactor même si le comportement reste identique. Voir [`../testing/`](../testing/) pour les principes généraux de test.

## 6. Commandes / syntaxe à connaître

```bash
npm create vite@latest mon-app -- --template react-ts   # créer un projet
npm run dev                                              # serveur de dev
npm run build                                             # build de production
npm run test                                               # lancer les tests (si configurés)
```

```tsx
const [state, setState] = useState(initial);
useEffect(() => { /* effet */ return () => { /* cleanup */ }; }, [deps]);
const memoValue = useMemo(() => compute(), [deps]);
const memoFn = useCallback(() => {}, [deps]);
const ref = useRef(initialValue);
const value = useContext(MyContext);
```

## 7. Exercices

Les énoncés sont dans `exercices/niveau-1.md`, `niveau-2.md`, `niveau-3.md`. Les corrections sont dans `solutions/` (mêmes noms de fichiers) — essaie de résoudre avant de regarder.

### Niveau 1 — Bases
Composants, props, `useState`, rendu conditionnel/listes.

### Niveau 2 — Intermédiaire
`useEffect` (fetch), hook personnalisé, formulaire contrôlé, Context.

### Niveau 3 — Avancé
Mémoïsation ciblée, Suspense + lazy loading, petit compound component.

## 8. Mini-projet

**Gestionnaire de tâches (Todo App) avec filtre et persistance locale.**

- Ajouter/supprimer/cocher des tâches (`useState`).
- Filtrer par statut (toutes/actives/terminées) via un state dérivé, pas via un effet.
- Persister la liste dans `localStorage` via un hook personnalisé `useLocalStorage`.
- Découper en composants (`TaskList`, `TaskItem`, `TaskFilter`) avec des `key` correctes.
- Bonus : ajouter React Router pour une page `/stats` séparée, et mémoïser le calcul des statistiques avec `useMemo`.

## Checklist

- [ ] Comprendre les fondamentaux (JSX, components, props, state)
- [ ] Savoir créer un projet React (Vite)
- [ ] Maîtriser la syntaxe principale des hooks de base
- [ ] Comprendre `useEffect` et ses pièges (dépendances, cleanup)
- [ ] Savoir debugger avec React DevTools
- [ ] Connaître les bonnes pratiques (composition, `key`, mémoïsation raisonnée)
- [ ] Réaliser les exercices
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (reconciliation, Server Components, Suspense)

## 10. Ressources

- [Documentation officielle React](https://react.dev) — référence à jour, inclut les hooks et les patterns modernes.
- [React Router](https://reactrouter.com) — documentation officielle du routeur.
- [Testing Library](https://testing-library.com/docs/react-testing-library/intro/) — philosophie et API de test.
- [roadmap.sh — React](https://roadmap.sh/react) — vue d'ensemble de l'écosystème.
- Next.js (framework full-stack basé sur React, non couvert dans ce dossier) : [nextjs.org/docs](https://nextjs.org/docs) — à explorer une fois React maîtrisé si besoin de SSR/RSC en production.
