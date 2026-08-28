# Solutions PHP — Niveau 3

## 1. Enum avec logique métier

```php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Cancelled = 'cancelled';

    public function canTransitionTo(self $next): bool
    {
        return match($this) {
            self::Pending => in_array($next, [self::Paid, self::Cancelled], true),
            self::Paid => in_array($next, [self::Shipped, self::Cancelled], true),
            self::Shipped, self::Cancelled => false, // états terminaux
        };
    }
}

var_dump(OrderStatus::Pending->canTransitionTo(OrderStatus::Paid));    // true
var_dump(OrderStatus::Shipped->canTransitionTo(OrderStatus::Pending)); // false — état terminal
```

## 2. Générateur pour un gros fichier

```php
function readOrders(string $path): \Generator
{
    $handle = fopen($path, 'r');
    if ($handle === false) {
        throw new \RuntimeException("Impossible d'ouvrir $path");
    }

    $header = fgetcsv($handle); // ["id", "total", "status"]

    while (($row = fgetcsv($handle)) !== false) {
        yield array_combine($header, $row);
    }

    fclose($handle);
}

function sumPaidTotal(iterable $orders): float
{
    $sum = 0.0;
    foreach ($orders as $order) {
        if ($order['status'] === 'paid') {
            $sum += (float) $order['total'];
        }
    }
    return $sum;
}

echo sumPaidTotal(readOrders('orders.csv'));
```

**Pourquoi la mémoire reste constante** : `yield` suspend l'exécution de la fonction après chaque ligne et rend la main à l'appelant — une seule ligne existe en mémoire à la fois. `file()` charge la totalité du fichier en mémoire d'un coup avant même de commencer à filtrer, ce qui explose avec un fichier de plusieurs millions de lignes.

## 3. Closures et composition de fonctions

```php
function pipe(callable ...$fns): callable
{
    return function ($input) use ($fns) {
        return array_reduce(
            $fns,
            fn($carry, $fn) => $fn($carry),
            $input
        );
    };
}

$process = pipe(
    fn(int $x) => $x + 1,
    fn(int $x) => $x * 2,
    fn(int $x) => $x - 3,
);
echo $process(5); // 9

$normalize = pipe(
    trim(...),
    strtolower(...),
    fn(string $s) => preg_replace('/\s+/', ' ', $s),
);
echo $normalize("  Hello   World  \n"); // "hello world"
```

## 4. Design orienté interface (mini refactor)

```php
interface PaymentMethod
{
    public function pay(array $order): void;
}

final class CardPayment implements PaymentMethod
{
    public function pay(array $order): void { /* logique carte */ }
}

final class PaypalPayment implements PaymentMethod
{
    public function pay(array $order): void { /* logique paypal */ }
}

final class TransferPayment implements PaymentMethod
{
    public function pay(array $order): void { /* logique virement */ }
}

final class OrderProcessor
{
    /** @var array<string, PaymentMethod> */
    private array $methods;

    public function __construct()
    {
        $this->methods = [
            'card' => new CardPayment(),
            'paypal' => new PaypalPayment(),
            'transfer' => new TransferPayment(),
        ];
    }

    public function process(array $order): void
    {
        $method = $this->methods[$order['payment_method']]
            ?? throw new \InvalidArgumentException("Mode de paiement inconnu : {$order['payment_method']}");

        $method->pay($order);
    }
}
```

**Gain** : ajouter un nouveau mode de paiement (ex. `crypto`) ne nécessite plus de modifier `OrderProcessor::process` — il suffit d'ajouter une classe implémentant `PaymentMethod` et une entrée dans le tableau de mapping (principe ouvert/fermé).
