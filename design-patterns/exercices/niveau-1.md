# Exercices — Niveau 1 : Bases

## 1. Identifier une violation du SRP

Voici une classe PHP :

```php
class Report {
    public function __construct(private array $data) {}

    public function calculateTotal(): float {
        return array_sum($this->data);
    }

    public function toHtml(): string {
        return "<p>Total: {$this->calculateTotal()}</p>";
    }

    public function sendByEmail(string $to): void {
        mail($to, "Rapport", $this->toHtml());
    }
}
```

Combien de responsabilités distinctes cette classe a-t-elle ? Liste-les, puis propose un découpage en plusieurs classes respectant le SRP.

## 2. Corriger une violation de l'Open/Closed Principle

Cette fonction JavaScript viole l'OCP :

```js
function getArea(shape) {
  if (shape.type === "circle") return Math.PI * shape.radius ** 2;
  if (shape.type === "square") return shape.side ** 2;
  // ajouter un triangle nécessite de modifier cette fonction
}
```

Réécris ce code pour qu'ajouter une nouvelle forme (ex. `triangle`) ne nécessite **aucune modification** de la fonction existante.

## 3. Nommage et Clean Code

Refactore ce snippet pour améliorer le nommage, sans changer le comportement :

```js
function f(d, r) {
  let x = d;
  if (r) x = x * 1.2;
  return x;
}
```

Que fait cette fonction ? Donne des noms qui rendent le code auto-explicatif, sans ajouter un seul commentaire.

## 4. Repérer un Singleton et ses inconvénients

```php
class Logger {
    private static ?Logger $instance = null;
    public static function getInstance(): Logger {
        if (self::$instance === null) {
            self::$instance = new Logger();
        }
        return self::$instance;
    }
    public function log(string $message): void { /* ... */ }
}
```

Cite deux problèmes concrets que ce Singleton pose pour tester une classe qui l'utilise. Propose une alternative.
