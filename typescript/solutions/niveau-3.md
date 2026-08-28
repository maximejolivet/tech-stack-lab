# Solutions — Niveau 3 : Avancé

## 1. Extraire le type résolu d'une Promise

```ts
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnwrapPromise<Promise<string>>; // string
type B = UnwrapPromise<number>;          // number
```

`infer U` capture le type interne de la Promise dans la branche `true` du type conditionnel. C'est exactement le mécanisme derrière le type natif `Awaited<T>` (en plus robuste, car il gère aussi les Promises imbriquées et les thenables).

## 2. Type JSON récursif

```ts
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };
```

Le type se référence lui-même dans ses deux dernières branches — TypeScript supporte la récursion dans les alias de type (avec des limites de profondeur/complexité au-delà desquelles le compilateur peut ralentir ou refuser).

## 3. API client typé de bout en bout

```ts
interface Endpoints {
  "/users": User[];
  "/users/:id": User;
  "/products": Product[];
}

async function apiGet<Path extends keyof Endpoints>(
  path: Path
): Promise<Endpoints[Path]> {
  const response = await fetch(path as string);
  return response.json();
}

const users = await apiGet("/users");       // typé User[]
const user = await apiGet("/users/:id");    // typé User
```

Le type de retour de `apiGet` est calculé automatiquement via `Endpoints[Path]` : impossible d'appeler `apiGet` avec un chemin qui n'existe pas dans `Endpoints`, et le type du résultat est toujours cohérent avec le chemin demandé — sans avoir à répéter le typage à chaque appel.

## 4. Validation runtime + typage dérivé

Le typage `Promise<User>` sur `getUser` est une **promesse non vérifiée** : `response.json()` retourne en réalité `any`, et l'assertion implicite via le type de retour de la fonction ne fait *aucune* vérification au runtime. Si l'API renvoie un champ manquant, un type différent, ou une erreur formatée en JSON, TS ne le détectera jamais — le bug se manifeste plus loin dans le code, là où la donnée est utilisée, rendant le diagnostic difficile.

Une lib comme Zod inverse la source de vérité : on définit un **schéma de validation** (`z.object({ id: z.number(), name: z.string(), email: z.string() })`), on l'exécute réellement sur la réponse (`UserSchema.parse(await response.json())`, qui throw si la donnée ne correspond pas), et le type TS `User` est **dérivé** du schéma (`z.infer<typeof UserSchema>`) plutôt que déclaré séparément. Le typage et la validation runtime ne peuvent alors plus diverger, et toute donnée malformée est détectée à la frontière du système (au moment du fetch), pas des couches plus tard.

## 5. Typage "brand" (nominal typing)

```ts
type Brand<T, B extends string> = T & { readonly __brand: B };

type UserId = Brand<number, "UserId">;
type ProductId = Brand<number, "ProductId">;

function getUser(id: UserId) { /* ... */ }

const userId = 1 as UserId;
const productId = 1 as ProductId;

getUser(userId);    // OK
getUser(productId);  // Erreur de compilation — bien que les deux soient des `number`
getUser(1);           // Erreur — un `number` brut n'est pas un `UserId`
```

La propriété `__brand` n'existe pas réellement au runtime (elle est effacée à la compilation) — c'est une astuce purement pour le compilateur, qui force à passer par une conversion explicite (`as UserId`) au lieu de laisser deux `number` sémantiquement différents s'échanger silencieusement.

## 6. Migration progressive

1. Renommer `tsconfig.json` en mode permissif : `"allowJs": true`, `"checkJs": false`, `"strict": false` dans un premier temps — le projet JS existant continue de fonctionner tel quel, TS coexiste avec les fichiers `.js`.
2. Typer les dépendances tierces sans types natifs via `@types/*` (DefinitelyTyped) quand ils existent, sinon une déclaration `.d.ts` minimale locale (`declare module "lib-sans-types";`) pour débloquer la compilation sans tout typer finement tout de suite.
3. Migrer fichier par fichier en renommant `.js` → `.ts` en commençant par les modules **sans dépendances internes** (utilitaires, types partagés) puis en remontant vers les points d'entrée — chaque renommage force à corriger les erreurs de typage locales à ce fichier seulement.
4. Activer `"checkJs": true` sur les fichiers `.js` restants pour bénéficier d'une vérification de type basique (via JSDoc) même avant leur migration complète.
5. Une fois 100% du code en `.ts`, activer progressivement `"strict": true` (ou directement si le budget le permet), puis retirer `allowJs`/`checkJs` devenus inutiles.
