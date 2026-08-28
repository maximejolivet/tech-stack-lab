# Solutions — Niveau 2 : Intermédiaire

## 1. Discriminated union

```ts
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function render<T>(state: RequestState<T>): string {
  switch (state.status) {
    case "idle": return "En attente";
    case "loading": return "Chargement...";
    case "success": return `Données : ${JSON.stringify(state.data)}`;
    case "error": return `Erreur : ${state.message}`;
    default: {
      const exhaustiveCheck: never = state; // erreur de compilation si un cas est oublié
      return exhaustiveCheck;
    }
  }
}
```

Le `default` avec `never` est un pattern courant pour forcer l'exhaustivité : si une variante est ajoutée à l'union sans être traitée dans le `switch`, TS refuse d'assigner `state` (qui ne serait plus `never`) à `exhaustiveCheck`.

## 2. Generic contraint

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "Max" };
getProperty(user, "id");   // number
getProperty(user, "name"); // string
// getProperty(user, "inexistant"); // Erreur : "inexistant" n'est pas une clé de `user`
```

## 3. Classe typée

```ts
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}
```

## 4. Utility types combinés

```ts
type ArticleDraft = Omit<Article, "id" | "publishedAt">;
type ArticleSummary = Pick<Article, "id" | "title" | "authorId">;
type PublishedArticle = Omit<Article, "publishedAt"> & { publishedAt: Date };
```

## 5. Function overloads

```ts
function createElement(tag: "img"): HTMLImageElement;
function createElement(tag: "div"): HTMLDivElement;
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}

const img = createElement("img"); // typé HTMLImageElement, accès direct à .src par ex.
const div = createElement("div"); // typé HTMLDivElement
```

## 6. Refactor vers `Record`

```ts
type Role = "admin" | "editor" | "viewer";
type RolePermissions = Record<Role, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};
```

`Record<Role, string[]>` garantit que **toutes** les clés de `Role` sont présentes (ni oubli, ni clé en trop) — une interface manuelle ne le vérifie pas automatiquement à l'ajout d'un nouveau rôle.
