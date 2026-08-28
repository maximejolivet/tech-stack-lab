# Solutions — Niveau 2 : Intermédiaire

## 1. Strategy pour les remises

```js
class StandardDiscount { apply(amount) { return amount; } }
class PremiumDiscount { apply(amount) { return amount * 0.9; } }
class VipDiscount { apply(amount) { return amount * 0.8; } }
class PartnerDiscount { apply(amount) { return amount * 0.85; } }

const discountStrategies = {
  standard: new StandardDiscount(),
  premium: new PremiumDiscount(),
  vip: new VipDiscount(),
  partner: new PartnerDiscount(),
};

function applyDiscount(customerType, amount) {
  const strategy = discountStrategies[customerType];
  if (!strategy) throw new Error(`Type de client inconnu: ${customerType}`);
  return strategy.apply(amount);
}
```

Ajouter un futur type de remise = ajouter une classe + une entrée dans `discountStrategies`, sans toucher `applyDiscount`.

## 2. Observer / event bus

```php
class EventBus {
    private array $listeners = [];

    public function on(string $event, callable $callback): void {
        $this->listeners[$event][] = $callback;
    }

    public function emit(string $event, mixed $payload): void {
        foreach ($this->listeners[$event] ?? [] as $callback) {
            $callback($payload);
        }
    }
}

$bus = new EventBus();
$bus->on('order.created', fn($order) => sendConfirmationEmail($order));
$bus->on('order.created', fn($order) => decrementStock($order));

$bus->emit('order.created', ['id' => 42, 'items' => [...]]);
```

Les deux abonnés réagissent indépendamment au même événement, sans se connaître ni connaître le code qui émet l'événement.

## 3. Interface Segregation

```ts
interface Authenticatable {
  login(): void;
  logout(): void;
}
interface UserManager {
  manageUsers(): void;
}
interface OrderViewer {
  viewOrders(): void;
}

class AdminAccount implements Authenticatable, UserManager, OrderViewer { /* ... */ }
class CustomerAccount implements Authenticatable, OrderViewer { /* ... */ }
class GuestAccount implements Authenticatable { /* ... */ }
```

Chaque compte n'implémente que les capacités qui le concernent réellement — plus de méthode vide ou d'exception "not supported".

## 4. Decorator empilable

```js
class Coffee {
  cost() { return 2; }
}
class MilkDecorator {
  constructor(coffee) { this.coffee = coffee; }
  cost() { return this.coffee.cost() + 0.5; }
}
class SugarDecorator {
  constructor(coffee) { this.coffee = coffee; }
  cost() { return this.coffee.cost() + 0.2; }
}

const order = new SugarDecorator(new MilkDecorator(new Coffee()));
order.cost(); // 2.7
```

Chaque décorateur enveloppe l'objet précédent et ajoute son propre coût — l'empilement se fait en composant les constructeurs, sans jamais modifier `Coffee`.
