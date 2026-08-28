# Exercices TypeScript — Niveau 2 : Intermédiaire

Objectif : combiner plusieurs notions (discriminated unions, generics contraints, utility types, classes typées).

## 1. Discriminated union

Modéliser un état de chargement de données avec une discriminated union `RequestState<T>` ayant 4 variantes : `idle`, `loading`, `success` (avec une propriété `data: T`), `error` (avec une propriété `message: string`). Écrire une fonction `render<T>(state: RequestState<T>): string` qui traite chaque cas de façon exhaustive (le compilateur doit signaler une erreur si un cas est oublié).

## 2. Generic contraint

Écrire une fonction générique `getProperty<T, K extends keyof T>(obj: T, key: K): T[K]` puis l'utiliser sur un objet `{ id: number; name: string }` pour récupérer `id` et `name` de façon typée. Vérifier (mentalement ou dans un playground) que `getProperty(obj, "inexistant")` ne compile pas.

## 3. Classe typée

Écrire une classe `Stack<T>` avec des méthodes `push(item: T): void`, `pop(): T | undefined`, `peek(): T | undefined`, et une propriété privée `items: T[]`. Ajouter une méthode `isEmpty(): boolean`.

## 4. Utility types combinés

Étant donné :

```ts
interface Article {
  id: number;
  title: string;
  content: string;
  authorId: number;
  publishedAt: Date | null;
}
```

Définir, sans dupliquer `Article` :
- `ArticleDraft` : comme `Article` mais sans `id` ni `publishedAt` (pour la création)
- `ArticleSummary` : uniquement `id`, `title`, `authorId`
- `PublishedArticle` : comme `Article` mais où `publishedAt` est garanti `Date` (non `null`)

## 5. Function overloads

Écrire une fonction `createElement` avec deux signatures possibles : `createElement("img"): HTMLImageElement` et `createElement("div"): HTMLDivElement`, avec une implémentation générique qui utilise `document.createElement`.

## 6. Refactor vers `Record`

Refactoriser ce code pour utiliser le type `Record` plutôt qu'une interface répétitive :

```ts
interface RolePermissions {
  admin: string[];
  editor: string[];
  viewer: string[];
}
```

en un type générique réutilisable pour n'importe quel ensemble de rôles (indice : combiner `Record` avec un type union de rôles).
