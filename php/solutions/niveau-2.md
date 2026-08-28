# Solutions PHP — Niveau 2

## 1. Classe avec property promotion

```php
final class Product
{
    public function __construct(
        public readonly string $name,
        public readonly float $price,
    ) {}

    public function priceWithTax(float $rate): float
    {
        return $this->price * (1 + $rate);
    }
}

$product = new Product("Clavier", 49.99);
echo $product->priceWithTax(0.2); // 59.988

$product->price = 10; // Error: Cannot modify readonly property Product::$price
```

## 2. Héritage et interfaces

```php
interface Discountable
{
    public function applyDiscount(float $percent): float;
}

final class Product implements Discountable
{
    public function __construct(private readonly float $price) {}

    public function applyDiscount(float $percent): float
    {
        return $this->price * (1 - $percent / 100);
    }
}

final class Service implements Discountable
{
    private const MIN_PRICE = 20.0;

    public function __construct(private readonly float $price) {}

    public function applyDiscount(float $percent): float
    {
        $discounted = $this->price * (1 - $percent / 100);
        return max($discounted, self::MIN_PRICE);
    }
}

/** @param Discountable[] $items */
function totalAfterDiscount(array $items, float $percent): float
{
    return array_sum(array_map(
        fn(Discountable $item) => $item->applyDiscount($percent),
        $items
    ));
}
```

L'intérêt : `totalAfterDiscount` ne connaît que le contrat `Discountable`, pas les classes concrètes — on peut ajouter un nouveau type d'item réductible sans toucher cette fonction (principe ouvert/fermé).

## 3. Exceptions métier

```php
final class InsufficientStockException extends \RuntimeException
{
    public function __construct(string $product, int $requested)
    {
        parent::__construct("Stock insuffisant pour \"$product\" : $requested unité(s) demandée(s)");
    }
}

function reserveStock(string $product, int $requested, int $available): void
{
    if ($requested > $available) {
        throw new InsufficientStockException($product, $requested);
    }
    // logique de réservation...
}

try {
    reserveStock("Clavier", 10, 3);
} catch (InsufficientStockException $e) {
    echo "Erreur : {$e->getMessage()}\n";
} finally {
    echo "Tentative de réservation loguée.\n"; // s'exécute que la réservation réussisse ou échoue
}
```

## 4. JSON et tableaux associatifs

```php
$json = '[{"id":1,"total":49.99,"status":"paid"},{"id":2,"total":120.00,"status":"pending"}]';

$orders = json_decode($json, true); // true = tableaux associatifs, pas des stdClass

$paidTotal = array_sum(array_map(
    fn($order) => $order['status'] === 'paid' ? $order['total'] : 0,
    $orders
));
// ou : array_sum(array_column(array_filter($orders, fn($o) => $o['status'] === 'paid'), 'total'));

$pending = array_values(array_filter($orders, fn($o) => $o['status'] === 'pending'));
$pendingJson = json_encode($pending);
```

## 5. Trait

```php
trait Timestampable
{
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;

    public function initTimestamps(): void
    {
        $this->createdAt = $this->updatedAt = new \DateTimeImmutable();
    }

    public function touch(): void
    {
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getUpdatedAt(): \DateTimeImmutable
    {
        return $this->updatedAt;
    }
}

final class Article
{
    use Timestampable;

    public function __construct(private readonly string $title)
    {
        $this->initTimestamps();
    }
}
```
