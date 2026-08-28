# TypeScript

## 1. Introduction

TypeScript est un **sur-ensemble de JavaScript** qui ajoute un système de types statique, compilé (transpilé) vers du JS standard. Ce dossier part du principe que [`../javascript/`](../javascript/) est acquis (closures, event loop, async/await...) et se concentre uniquement sur ce que TypeScript **ajoute** : annotations de types, inférence, generics, narrowing, et l'outillage associé (`tsconfig.json`).

**À quoi sert-il ?**
- Détecter des bugs de typage **à la compilation**, avant l'exécution (accès à une propriété inexistante, mauvais type d'argument, `null` non géré).
- Documenter le code de façon vérifiable : les types servent de contrat explicite entre les fonctions/modules.
- Fiabiliser le refactoring à grande échelle (renommer une propriété casse la compilation partout où elle est utilisée, au lieu d'un bug silencieux en prod).

**Où se situe-t-il dans une architecture web ?**
Partout où il y a du JS : front (React/Vue/Angular en TS par défaut aujourd'hui), backend (Node.js/NestJS), scripts et tooling. Le typage est **effacé à la compilation** — TypeScript n'existe plus au runtime, ce n'est qu'une aide au développement.

**Avantages**
- Erreurs détectées avant l'exécution plutôt qu'en production.
- Autocomplétion et navigation de code bien plus fiables dans l'IDE.
- Facilite la collaboration sur une base de code partagée (le type est une documentation qui ne peut pas mentir, contrairement à un commentaire).

**Limites**
- Coût d'apprentissage et de configuration (`tsconfig.json`, résolution de types de librairies tierces).
- Le typage n'est **pas une garantie runtime** : `as` et `any` permettent de mentir au compilateur ; les données externes (API, `JSON.parse`) ne sont pas validées par TS seul (il faut une lib comme Zod pour une validation réelle).
- Étape de build supplémentaire (sauf environnements avec support natif type-stripping comme Node.js récent ou Deno).

## 2. Prérequis

- Maîtriser [`../javascript/`](../javascript/) : ce dossier ne réexplique pas les bases JS (fonctions, scope, async/await, DOM...), uniquement la couche de typage.
- Node.js et npm installés (voir [`../nodejs/`](../nodejs/)) pour exécuter le compilateur `tsc`.

## 3. Rappel des bases 🟢

### 01 - Types primitifs et littéraux

**Explication** — Les types de base reprennent les types JS (`string`, `number`, `boolean`, `null`, `undefined`), plus les **types littéraux** (une valeur précise comme type) et `void` (absence de retour utile).

```ts
let name: string = "Max";
let age: number = 30;
let active: boolean = true;

let status: "pending" | "done" = "pending"; // type littéral : seules ces 2 valeurs sont valides
status = "cancelled"; // Erreur de compilation

function log(message: string): void {
  console.log(message); // ne retourne rien d'exploitable
}
```

**Cas d'usage** : les littéraux remplacent avantageusement des "magic strings" par un ensemble fermé de valeurs vérifiées à la compilation (états, rôles, statuts).

**Erreur fréquente** : annoter un type déjà évident par inférence (`let age: number = 30`) — verbeux et redondant, TS l'infère seul.

**Bonne pratique** : laisser l'inférence faire son travail sur les déclarations simples ; annoter explicitement les paramètres de fonction, les retours de fonction publique, et les cas ambigus.

### 02 - Typage des fonctions

**Explication** — On type les paramètres et, si besoin, le retour. TS infère souvent le retour tout seul.

```ts
function add(a: number, b: number): number {
  return a + b;
}

const multiply = (a: number, b: number): number => a * b;

function greet(name: string, greeting = "Bonjour"): string { // valeur par défaut → type inféré
  return `${greeting}, ${name}`;
}

function sum(...numbers: number[]): number { // rest typé
  return numbers.reduce((acc, n) => acc + n, 0);
}

type Callback = (error: Error | null, result?: string) => void; // type pour une fonction callback
```

**Erreur fréquente** : typer un paramètre optionnel comme `nullable` (`name: string | null`) alors qu'on veut un paramètre **absent** (`name?: string`) — ce sont deux contrats différents (absent vs présent-mais-null).

**Bonne pratique** : toujours typer les paramètres (l'inférence ne peut pas deviner l'intention de l'appelant) ; laisser le retour s'inférer sauf API publique d'une librairie (où l'expliciter fige le contrat et accélère la compilation).

### 03 - Interfaces vs Type Aliases

**Explication** — Deux syntaxes pour nommer une forme d'objet, très proches en pratique.

```ts
interface User {
  id: number;
  name: string;
  email?: string; // propriété optionnelle
  readonly createdAt: Date; // non réassignable après création
}

type UserType = {
  id: number;
  name: string;
};
```

**Différences clés** : `interface` peut être **étendue** par déclaration multiple (declaration merging, utile pour augmenter un type de librairie tierce) et utilise `extends` pour l'héritage ; `type` peut représenter des unions, intersections, tuples, et types utilitaires — ce qu'`interface` ne peut pas faire seule.

```ts
interface Admin extends User { role: "admin"; } // héritage d'interface
type ID = string | number;                       // union — impossible avec interface
```

**Erreur fréquente** : croire qu'il faut choisir une seule convention pour tout le projet sans raison technique — mélanger les deux au hasard nuit à la cohérence.

**Bonne pratique** (consensus actuel) : `interface` pour les formes d'objets/classes publiques (API, props de composants) ; `type` dès qu'il faut une union, une intersection, ou un alias sur un type primitif/complexe.

### 04 - Union & Intersection

**Explication** — `|` (union) : une valeur est **l'un** des types listés. `&` (intersection) : une valeur doit satisfaire **tous** les types listés à la fois.

```ts
type ID = string | number;                 // union
function printId(id: ID) { console.log(id); }

type Timestamped = { createdAt: Date };
type Named = { name: string };
type Entity = Timestamped & Named;          // intersection : doit avoir les deux
const entity: Entity = { createdAt: new Date(), name: "Max" };
```

**Cas d'usage** : union pour modéliser un état parmi plusieurs (ex. `"idle" | "loading" | "success" | "error"`) ; intersection pour composer plusieurs formes (mixins de propriétés).

**Erreur fréquente** : accéder à une propriété spécifique à un seul membre d'une union sans avoir fait de narrowing au préalable → erreur de compilation (voir notion suivante).

**Bonne pratique** : préférer une union de types littéraux à un `enum` ou un `boolean` pour modéliser un état à plus de 2 valeurs (plus lisible, extensible sans casser l'existant).

### 05 - Narrowing (type guards)

**Explication** — Réduire (« affiner ») le type d'une variable au sein d'un bloc conditionnel, pour que TS sache précisément quel membre de l'union est actif.

```ts
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase(); // TS sait ici que id est `string`
  }
  return id.toFixed(2); // TS sait ici que id est `number`
}

interface Circle { kind: "circle"; radius: number; }
interface Square { kind: "square"; side: number; }
type Shape = Circle | Square;

function area(shape: Shape): number { // discriminated union
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "square": return shape.side ** 2;
  }
}
```

**Erreur fréquente** : oublier un cas dans un `switch` sur une union discriminée — TS ne le signale pas par défaut sans `strict` + un `default` qui exploite `never` (voir plus bas).

**Bonne pratique** : modéliser les états mutuellement exclusifs avec une **discriminated union** (propriété commune type `kind`/`type`) plutôt qu'un objet avec plein de champs optionnels — le compilateur garantit alors l'exhaustivité du traitement.

### 06 - `any` vs `unknown` vs `never`

**Explication** — Trois types spéciaux souvent confondus.

```ts
let a: any = "hello";
a.toFixed();      // aucune erreur de compilation... mais crash au runtime !

let u: unknown = "hello";
u.toUpperCase();       // Erreur de compilation : il faut d'abord vérifier le type
if (typeof u === "string") {
  u.toUpperCase();     // OK, narrowing fait
}

function fail(message: string): never { // ne retourne jamais (throw ou boucle infinie)
  throw new Error(message);
}
```

`any` désactive complètement la vérification de type (dette technique). `unknown` est le équivalent **type-safe** : on peut y assigner n'importe quoi, mais on doit vérifier le type avant de l'utiliser. `never` représente une valeur qui ne peut jamais exister (fonction qui throw toujours, branche impossible).

**Erreur fréquente** : typer une réponse d'API externe en `any` par facilité — perd tout le bénéfice de TS sur cette donnée et sur tout ce qui en dérive.

**Bonne pratique** : bannir `any` en pratique professionnelle (activer `noImplicitAny` et éviter `any` explicite) ; utiliser `unknown` pour toute donnée dont le type n'est pas garanti à la compilation (réponse HTTP, `JSON.parse`, entrée utilisateur), puis valider/narrower avant usage.

### 07 - Arrays & Tuples

**Explication** — Un array typé contient des éléments d'un seul type ; un tuple est un array de **longueur et types fixes** par position.

```ts
const names: string[] = ["Max", "Alex"];
const scores: Array<number> = [10, 20, 30]; // syntaxe générique équivalente

type Point = [number, number];         // tuple : exactement 2 nombres
const origin: Point = [0, 0];

type Entry = [key: string, value: number]; // tuple nommé (lisibilité)
```

**Erreur fréquente** : utiliser un array classique là où un tuple exprimerait mieux une structure fixe (ex. `[string, number]` pour une paire clé/valeur), perdant la vérification de position et de longueur.

**Bonne pratique** : tuple pour une structure de taille fixe et hétérogène connue (coordonnées, paire), array typé pour une collection homogène de taille variable.

### 08 - Generics

**Explication** — Un generic est un "type paramétré" : la fonction/type reste réutilisable pour n'importe quel type concret, tout en gardant la relation entre entrée et sortie vérifiée par le compilateur.

```ts
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}
firstElement([1, 2, 3]);        // T inféré = number, retour : number | undefined
firstElement(["a", "b"]);       // T inféré = string, retour : string | undefined

interface Box<T> {
  value: T;
}
const numberBox: Box<number> = { value: 42 };

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] { // contrainte generic
  return obj[key];
}
```

**Cas d'usage** : fonctions utilitaires réutilisables (`firstElement`, `groupBy`), wrappers génériques (`ApiResponse<T>`, `Box<T>`), hooks/composables typés.

**Erreur fréquente** : sur-généraliser une fonction qui n'a en réalité qu'un seul type d'usage réel dans le code — complexité inutile sans bénéfice.

**Bonne pratique** : introduire un generic seulement quand une fonction/type doit rester correcte pour plusieurs types concrets tout en préservant la relation entre eux ; contraindre avec `extends` dès que possible plutôt que laisser un generic totalement libre.

### 09 - Enums

**Explication** — Ensemble nommé de constantes. TypeScript propose aussi `as const` comme alternative plus légère, souvent préférée aujourd'hui.

```ts
enum Role {
  Admin = "ADMIN",
  Editor = "EDITOR",
  Viewer = "VIEWER",
}
const r: Role = Role.Admin;

// Alternative "as const" — pas de code généré à la compilation, juste des types
const ROLES = ["ADMIN", "EDITOR", "VIEWER"] as const;
type RoleConst = typeof ROLES[number]; // "ADMIN" | "EDITOR" | "VIEWER"
```

**Erreur fréquente** : utiliser un `enum` numérique par défaut (`enum Role { Admin, Editor }`) — les valeurs (`0`, `1`...) sont peu lisibles en dehors du code TS (logs, JSON, DB).

**Bonne pratique** : préférer un `enum` à valeurs `string` explicites, ou une union de littéraux via `as const` (plus léger, pas de code JS généré, meilleure interopérabilité JSON). Beaucoup d'équipes évitent `enum` par défaut au profit de `as const` pour cette raison.

### 10 - Utility Types

**Explication** — Types génériques fournis nativement par TypeScript pour transformer un type existant sans le réécrire.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;        // toutes les propriétés optionnelles (ex. formulaire d'édition)
type UserPreview = Pick<User, "id" | "name">; // ne garde que certaines propriétés
type UserWithoutEmail = Omit<User, "email">;  // toutes sauf certaines
type ReadonlyUser = Readonly<User>;      // toutes les propriétés en lecture seule
type UsersById = Record<number, User>;   // dictionnaire clé/valeur typé
type RequiredUser = Required<PartialUser>; // rend toutes les propriétés obligatoires
```

**Cas d'usage** : `Partial` pour un payload de mise à jour partielle (PATCH), `Pick`/`Omit` pour dériver un DTO d'un modèle sans duplication, `Record` pour typer un objet utilisé comme map.

**Erreur fréquente** : redéfinir manuellement un type dérivé d'un type existant (ex. recopier `User` sans `email`) au lieu d'utiliser `Omit<User, "email">` — les deux types divergent silencieusement au premier changement du type source.

**Bonne pratique** : dériver systématiquement les types depuis une seule source de vérité avec les utility types, plutôt que dupliquer des définitions proches.

### 11 - Assertions de type

**Explication** — Affirmer au compilateur qu'une valeur a un type précis, sans conversion réelle au runtime. À utiliser avec prudence : c'est vous qui prenez la responsabilité, pas TS.

```ts
const input = document.querySelector("#email") as HTMLInputElement; // TS ne peut pas le déduire seul
console.log(input.value);

const config = {} as AppConfig; // dangereux : objet vide affirmé comme AppConfig complet

const el = document.querySelector("#email")!; // "!" (non-null assertion) : affirme que ce n'est pas null
```

**Erreur fréquente** : utiliser `as` pour faire taire une erreur de compilation légitime plutôt que corriger le vrai problème de typage — bug potentiel déplacé au runtime, invisible pour TS.

**Bonne pratique** : réserver `as` aux cas où TS ne peut objectivement pas connaître le type précis (résultat de `querySelector`, résultat externe validé autrement) ; préférer un vrai type guard/narrowing dès que c'est possible plutôt qu'une assertion.

### 12 - Modules & déclarations (`.d.ts`)

**Explication** — Le système `import`/`export` est celui de JS ESM (voir [`../javascript/`](../javascript/)). Les fichiers `.d.ts` contiennent uniquement des **déclarations de types**, sans implémentation — utilisés pour typer une librairie JS sans types natifs, ou exposer les types d'un module.

```ts
// types/global.d.ts
declare module "*.svg" {
  const content: string;
  export default content;
}

// api.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
}
export function fetchUser(id: number): Promise<ApiResponse<User>> { /* ... */ }
```

**Erreur fréquente** : ignorer une erreur "Could not find a declaration file for module 'x'" en installant `any` partout plutôt que chercher le paquet `@types/x` (DefinitelyTyped) ou écrire une déclaration minimale.

**Bonne pratique** : privilégier les librairies avec types natifs ou un paquet `@types/*` officiel ; centraliser les déclarations custom dans un dossier `types/` dédié du projet.

### 13 - `tsconfig.json` et niveaux de strictness

**Explication** — Fichier de configuration du compilateur. Le niveau de rigueur du typage dépend directement des options activées.

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "strict": true,              // active tout le paquet de vérifications strictes
    "noUncheckedIndexedAccess": true, // accès à un tableau/objet par index retourne T | undefined
    "noImplicitAny": true,       // interdit un `any` implicite non annoté
    "esModuleInterop": true
  }
}
```

`strict: true` active en une seule option `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, et plusieurs autres — c'est le réglage recommandé en toutes circonstances professionnelles.

**Erreur fréquente** : démarrer un projet avec `strict: false` "pour aller plus vite", puis devoir migrer un gros projet vers le mode strict plus tard (coût largement supérieur à l'avoir activé dès le départ).

**Bonne pratique** : toujours `"strict": true` sur un nouveau projet ; activer `noUncheckedIndexedAccess` (non inclus dans `strict`) qui évite un piège fréquent : accéder à `arr[i]` est typé `T`, pas `T | undefined`, par défaut — faux sentiment de sécurité sur les accès hors limites.

## 4. Concepts intermédiaires 🟡

- **Types conditionnels** : `T extends U ? X : Y` — logique de type calculée selon une condition, base de nombreux utility types avancés.
- **Types mappés** : générer un type en itérant sur les clés d'un autre (`{ [K in keyof T]: ... }`) — c'est ainsi que `Partial`/`Readonly` sont eux-mêmes implémentés en interne.
- **`keyof` et lookup types** : `keyof T` donne l'union des clés de `T` ; `T[K]` accède au type d'une propriété précise. Combinaison essentielle pour des fonctions génériques sûres (voir `getProperty` plus haut).
- **Function overloads** : plusieurs signatures possibles pour une même fonction, utile quand le type de retour dépend du type d'argument d'une façon que les generics seuls n'expriment pas bien.
- **Typage des classes** : modificateurs d'accès (`private`/`protected`/`public`), propriétés en paramètre de constructeur (raccourci `constructor(private name: string)`), interfaces implémentées par une classe (`implements`).
- **Module augmentation** : étendre le typage d'un module tiers existant (ex. ajouter une propriété custom à `Express.Request`) via declaration merging.
- **Debugging de types** : lire un message d'erreur TS complexe en partant de l'intérieur (le type le plus imbriqué), utiliser `// @ts-expect-error` pour documenter un cas volontairement non typé, hover dans l'IDE pour voir le type inféré réel.
- **Intégration avec les frameworks** : props typées (React `FC<Props>`, Vue `defineProps<Props>()`), typage des stores d'état, typage de bout en bout avec une API (client généré depuis un schéma OpenAPI/GraphQL) — chaque framework a son propre dossier dans ce repo pour le détail.

## 5. Concepts avancés 🟠🔴

- **Types récursifs** : un type qui se référence lui-même (ex. `type Json = string | number | boolean | null | Json[] | { [key: string]: Json }`), utile pour modéliser des structures arbitrairement imbriquées.
- **Inférence dans les generics complexes** : `infer` dans un type conditionnel pour extraire un type imbriqué (ex. extraire le type résolu d'une `Promise<T>`).
- **Types "brand"/nominal typing** : TypeScript est structurellement typé par défaut (deux types avec la même forme sont interchangeables) ; simuler un typage nominal (ex. distinguer `UserId` de `ProductId`, tous deux des `string`) via une propriété fantôme.
- **Validation runtime vs typage compile-time** : TS ne protège jamais contre une donnée externe malformée (API, form, fichier) — nécessite une lib de validation (Zod, Valibot) qui **dérive** le type TS du schéma de validation, gardant une seule source de vérité.
- **Performance de compilation** : sur un gros monorepo, un typage trop complexe (types récursifs profonds, unions énormes) peut ralentir significativement `tsc` — savoir simplifier ou isoler les types coûteux.
- **Architecture de types à grande échelle** : partager des types entre front et back (monorepo, package `@types` interne, ou génération depuis un schéma API), éviter la duplication de contrats entre client et serveur.
- **Migration progressive JS → TS** : `allowJs` + `checkJs` pour typer un projet JS existant graduellement, fichier par fichier, sans tout migrer d'un coup.

## 6. Commandes / syntaxe à connaître

```bash
npm install -D typescript          # installer TS en dépendance de dev
npx tsc --init                     # générer un tsconfig.json de base
npx tsc                            # compiler selon tsconfig.json
npx tsc --watch                    # recompiler à chaque sauvegarde
npx tsc --noEmit                   # vérifier les types sans générer de fichiers JS (usage CI)
npx ts-node src/index.ts           # exécuter un fichier .ts directement (dev/scripts)
```

```ts
// Syntaxe essentielle à avoir sous les doigts
function fn(a: string, b?: number): void {}
type T = A | B;
interface I { a: string; b?: number; readonly c: boolean; }
function generic<T extends object>(x: T): T { return x; }
const x = value as SpecificType;
type Keys = keyof SomeType;
type Partial2 = Partial<SomeType>;
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Client API typé pour une todo-list**

Reprendre le mini-projet todo-list de [`../javascript/`](../javascript/) et le porter en TypeScript avec :
- Un modèle `Task` (interface) et un état d'application typé via une discriminated union (`{ status: "idle" } | { status: "loading" } | { status: "success"; tasks: Task[] } | { status: "error"; message: string }`).
- Des fonctions API (`fetchTasks`, `createTask`, `updateTask`) génériques sur un type `ApiResponse<T>`, avec gestion des erreurs typée (classe d'erreur custom).
- Un DTO de création dérivé du modèle `Task` via `Omit`/`Pick` plutôt que redéfini à la main.
- `tsconfig.json` en mode `strict` complet, zéro `any` dans le code, zéro warning à la compilation (`tsc --noEmit`).

Objectif : voir concrètement comment le typage change la façon d'écrire du code déjà familier en JS, et où il attrape des bugs qui seraient passés inaperçus en vanilla JS.

## Checklist

- [ ] Comprendre les fondamentaux (types primitifs, interfaces, unions, generics)
- [ ] Savoir créer un projet TS (tsconfig, compilation, `strict` activé)
- [ ] Maîtriser la syntaxe principale (narrowing, utility types, tuples)
- [ ] Comprendre les concepts importants (types conditionnels/mappés, `keyof`, classes typées)
- [ ] Savoir debugger (lire les erreurs de type, `@ts-expect-error`, inspection IDE)
- [ ] Connaître les bonnes pratiques (bannir `any`, dériver les types plutôt que dupliquer)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (types récursifs, `infer`, validation runtime, architecture de types)

## 10. Ressources

- [TypeScript Handbook (officiel)](https://www.typescriptlang.org/docs/handbook/intro.html) — référence complète et à jour.
- [TypeScript — Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html) — liste exhaustive des utility types natifs.
- [TS Playground](https://www.typescriptlang.org/play) — tester du code TS et voir le JS généré, sans setup local.
- [Total TypeScript (Matt Pocock)](https://www.totaltypescript.com/) — ressource reconnue pour les patterns de types avancés.
- [roadmap.sh — TypeScript](https://roadmap.sh/typescript) — vue d'ensemble du parcours d'apprentissage.
