# Exercices TypeScript — Niveau 3 : Avancé

Objectif : problèmes proches de situations professionnelles (types conditionnels, `infer`, architecture de types, validation runtime).

## 1. Extraire le type résolu d'une Promise

Sans utiliser le type utilitaire natif `Awaited<T>`, écrire un type conditionnel `UnwrapPromise<T>` qui extrait le type `T` d'une `Promise<T>` (et retourne `T` tel quel si ce n'est pas une Promise), en utilisant `infer`.

```ts
type A = UnwrapPromise<Promise<string>>; // doit donner "string"
type B = UnwrapPromise<number>;          // doit donner "number"
```

## 2. Type JSON récursif

Définir un type `JSONValue` représentant n'importe quelle valeur JSON valide (string, number, boolean, null, tableau de `JSONValue`, ou objet dont les valeurs sont des `JSONValue`).

## 3. API client typé de bout en bout

Concevoir les types pour un petit client API avec :
- Une interface `Endpoints` qui mappe un chemin d'API à son type de réponse (ex. `"/users": User[]`, `"/users/:id": User`).
- Une fonction générique `apiGet<Path extends keyof Endpoints>(path: Path): Promise<Endpoints[Path]>` dont le type de retour dépend automatiquement du chemin passé en argument.

## 4. Validation runtime + typage dérivé

En pseudo-code (sans installer de librairie), expliquer par écrit pourquoi ce code est dangereux malgré le typage TS, et comment une lib comme Zod résoudrait le problème en dérivant le type TS d'un schéma de validation plutôt que l'inverse :

```ts
interface User { id: number; name: string; email: string; }

async function getUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json(); // typé User... mais est-ce garanti ?
}
```

## 5. Typage "brand" (nominal typing)

TypeScript est structurellement typé : `type UserId = number` et `type ProductId = number` sont interchangeables, ce qui permet de passer un `ProductId` là où un `UserId` est attendu sans erreur. Proposer une technique pour rendre `UserId` et `ProductId` non interchangeables entre eux, tout en restant basés sur `number`.

## 6. Migration progressive

Un projet JS existant de taille moyenne doit migrer vers TypeScript sans tout réécrire d'un coup. Décrire une stratégie concrète en 4-5 étapes (options de `tsconfig.json` à utiliser, ordre de migration des fichiers, comment gérer les dépendances tierces sans types).
