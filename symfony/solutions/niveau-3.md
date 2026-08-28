# Solutions — Niveau 3

## Exercice 1

```php
class ArticleVoter extends Voter
{
    protected function supports(string $attribute, mixed $subject): bool
    {
        return in_array($attribute, ['EDIT', 'DELETE']) && $subject instanceof Article;
    }

    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();
        if (!$user instanceof User) {
            return false;
        }

        if (in_array('ROLE_ADMIN', $user->getRoles(), true)) {
            return true;
        }

        return $subject->getAuthor() === $user;
    }
}
```

```php
#[Route('/articles/{id}/edit')]
public function edit(Article $article): Response
{
    $this->denyAccessUnlessGranted('EDIT', $article);
    // ...
}
```

## Exercice 2

```php
final class OrderPlacedEvent
{
    public function __construct(public readonly int $orderId) {}
}
```

```php
$this->eventDispatcher->dispatch(new OrderPlacedEvent($order->getId()));
```

```php
class OrderPlacedSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [OrderPlacedEvent::class => 'onOrderPlaced'];
    }

    public function onOrderPlaced(OrderPlacedEvent $event): void
    {
        $this->logger->info("Commande #{$event->orderId} passée");
    }
}
```

Pour rendre l'action asynchrone (ex. envoi d'un email de confirmation, génération de facture PDF — traitement lent qui ne doit pas bloquer la réponse HTTP), on remplace le Subscriber par un **message** dédié dispatché sur le bus Messenger (`$this->bus->dispatch(new SendOrderConfirmation($order->getId()))`) traité par un `Handler` consommé en arrière-plan (`messenger:consume async`). Un Event Subscriber reste synchrone dans la même requête ; Messenger découple l'exécution dans le temps et permet de réessayer en cas d'échec (retry policy).

## Exercice 3

```text
src/
├── Domain/
│   ├── Cart.php                 # entité métier pure, aucune dépendance Doctrine
│   ├── CartItem.php
│   └── CartRepositoryInterface.php   # interface définie ici
├── Application/
│   └── AddItemToCartHandler.php # cas d'usage, orchestre le Domain
└── Infrastructure/
    └── Doctrine/
        └── DoctrineCartRepository.php  # implémente CartRepositoryInterface avec Doctrine
```

Le `Domain` ne doit dépendre d'aucune classe Doctrine/Symfony pour rester testable unitairement sans base de données et réutilisable si l'infrastructure change (ex. migration vers un autre ORM). L'interface `CartRepositoryInterface::save(Cart $cart): void` est définie dans `Domain`, son implémentation concrète `DoctrineCartRepository` vit dans `Infrastructure` et est injectée via le service container (autowiring sur l'interface, alias configuré vers l'implémentation).

## Exercice 4

- **`APP_ENV=prod`** : désactive le profiler/debug toolbar (fuite d'informations, overhead) et active le cache de configuration compilé.
- **`opcache.validate_timestamps=0`** : évite de revérifier le hash de chaque fichier PHP à chaque requête — oublié, chaque requête paie un coût disque inutile ; en contrepartie il faut vider opcache à chaque déploiement.
- **`php bin/console cache:warmup`** : précompile le container DI et le routing avant la première requête — oublié, la première requête après déploiement (ou chaque requête si le cache est invalidé en boucle) paie le coût de compilation.
- **Secrets via `secrets:set` / vault Symfony** plutôt qu'en clair dans `.env` sur le serveur — oublié, risque de fuite de credentials si le serveur ou le repo de déploiement est compromis.
