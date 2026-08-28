# Solutions — Niveau 1 : Bases

## 1. Violation du SRP

Trois responsabilités : calcul métier (`calculateTotal`), présentation (`toHtml`), et envoi (`sendByEmail`). Trois raisons de changer indépendantes (une règle de calcul change, le format d'affichage change, le mécanisme d'envoi change).

```php
class Report {
    public function __construct(private array $data) {}
    public function calculateTotal(): float {
        return array_sum($this->data);
    }
}

class ReportHtmlRenderer {
    public function render(Report $report): string {
        return "<p>Total: {$report->calculateTotal()}</p>";
    }
}

class ReportMailer {
    public function send(string $html, string $to): void {
        mail($to, "Rapport", $html);
    }
}
```

## 2. Corriger l'Open/Closed Principle

```js
class Circle {
  constructor(radius) { this.radius = radius; }
  area() { return Math.PI * this.radius ** 2; }
}
class Square {
  constructor(side) { this.side = side; }
  area() { return this.side ** 2; }
}
class Triangle {
  constructor(base, height) { this.base = base; this.height = height; }
  area() { return (this.base * this.height) / 2; }
}

function getArea(shape) {
  return shape.area();
}
```

Ajouter `Triangle` (ou n'importe quelle forme future) ne touche pas `getArea` : chaque forme porte sa propre logique de calcul derrière une méthode commune `area()`.

## 3. Nommage et Clean Code

```js
function applyLoyaltyBonus(baseAmount, isRegularCustomer) {
  let finalAmount = baseAmount;
  if (isRegularCustomer) {
    finalAmount = finalAmount * 1.2;
  }
  return finalAmount;
}
```

La fonction applique un bonus de fidélité de 20% pour un client régulier. Les noms explicites (`baseAmount`, `isRegularCustomer`, `applyLoyaltyBonus`) rendent la logique lisible sans commentaire.

## 4. Singleton et ses inconvénients

Deux problèmes concrets :
1. **Impossible d'isoler un test** : `Logger::getInstance()` retourne toujours la même instance entre deux tests, donc l'état (ex. buffer de logs) fuit d'un test à l'autre si on ne le réinitialise pas manuellement.
2. **Dépendance cachée** : une classe qui appelle `Logger::getInstance()` en interne ne déclare pas cette dépendance dans son constructeur — impossible de la remplacer par un mock sans modifier le code de la classe elle-même.

Alternative : injecter une interface `LoggerInterface` par constructeur, et laisser un conteneur d'injection de dépendances (ou, à défaut, le point d'entrée de l'application) décider de fournir une instance partagée. Le comportement "instance unique" est alors une décision de configuration du conteneur, pas une contrainte figée dans la classe elle-même.
