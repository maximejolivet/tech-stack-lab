# Solutions — Niveau 1 : Bases

## 1. Typer une fonction

```ts
function calculateTotal(price: number, quantity: number, taxRate: number): number {
  return price * quantity * (1 + taxRate);
}
```

## 2. Interface

```ts
interface Product {
  id: number;
  name: string;
  price: number;
  description?: string;
  readonly inStock: boolean;
}
```

## 3. Union de littéraux

```ts
type OrderStatus = "pending" | "shipped" | "delivered" | "cancelled";

function isFinal(status: OrderStatus): boolean {
  return status === "delivered" || status === "cancelled";
}
```

## 4. Narrowing

```ts
function formatValue(value: string | number | boolean): string {
  if (typeof value === "string") return value.toUpperCase();
  if (typeof value === "number") return value.toFixed(2);
  return value ? "oui" : "non";
}
```

## 5. `any` vs `unknown`

```ts
function processInput(input: unknown): string {
  if (typeof input === "string") {
    return input.toUpperCase();
  }
  throw new Error("processInput attend une chaîne de caractères");
}
```

`unknown` force à vérifier le type avant utilisation — le compilateur refuse `input.toUpperCase()` tant qu'aucun narrowing n'a été fait, contrairement à `any` qui laisse passer l'erreur jusqu'au runtime.

## 6. Generic simple

```ts
function wrapInArray<T>(value: T): T[] {
  return [value];
}

wrapInArray(42);      // number[]
wrapInArray("hello"); // string[]
```

## 7. Utility type

```ts
type ProductPreview = Pick<Product, "id" | "name">;
type ProductUpdate = Partial<Product>;
```
